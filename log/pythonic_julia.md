# Julia performance & dot access pattern

APIs in Julia sometimes feel a bit clunky - obviously this is subjective, but being used to python, `object.service()` feels cleaner than `service(object)` since `object` can hide details of how `service` is implemented, but with `service(object)`, `service` must understand the internals of `object`. This is very `OOP` - encapsulation, and this issue of universal function call syntax has been discussed: [issue:31779](https://github.com/JuliaLang/julia/issues/31779), [pr:25052](https://github.com/JuliaLang/julia/pull/25052), and shut down as julia is not OOP. However, rust has dot call syntax, and it isn't OOP. 

Here's another hacky attempt to get `object.service()` in Julia for cleaner APIs, while being explicit about performance overheads. 

## The Closure/Capture Method (Mimicking Python)

You can mimic Python's `self` by defining functions inside a constructor that capture local variables.

```julia
struct MockPythonMethod
   data
   f
end

function make()::MockPythonMethod
    local data::Vector{Int} = zeros(10)
    function apply()
        a = data
        for i in eachindex(a)
            a[i] += 1
        end
    end
    return MockPythonMethod(data, apply)
end

```

There's a performance risk: in `y1 = make(); y2 = make();`. The access `y1.f` is dynamic since `f` is dynamically typed, though the call `y1.f()` is optimized. This may matter in a hot loop, but ideally with jit, this gets optimized away. Inspecting with `@code_warntype` shows that the type is `Any` on the lookup.  

```
@code_warntype y1.f
MethodInstance for getproperty(::MockPythonMethod, ::Symbol)
  from getproperty(x, f::Symbol) @ Base Base_compiler.jl:54
Arguments
  #self#::Core.Const(getproperty)
  x::MockPythonMethod
  f::Symbol
Body::Any
1 ─      nothing
│   %2 = Base.getfield::Core.Const(getfield)
│   %3 = (%2)(x, f)::Any
└──      return %3
```

One possible way to avoid this is to parametrize the types  'data::DType', 'f::F' with `MockPythonMethod{DType, F}`. 

In `@code_warntype obj.f()`, any red text or `Any`, `Core.Box`, all indicate captured variables are not statically typed. This needs to be avoided to ensure the code avoids multiple dispatch and compiles to optimized native machine code. Now to verify type stability, in `dump(obj.f)`, the closure's hidden fields must all be types to be type stable (e.g., `Vector{Int64}`). Looking at actual native code, `@code_native obj.f()`, seeing `jl_apply_generic` means dynamically typed fields.

---

## Type-Stable Multiple Methods

Instead of putting dynamically typed functions inside the object, we define functions that take the object as an argument, which is the julia way:

### The getproperty pattern

```julia
struct TestObject
    data::Vector{Int}
end

function increment!(self::TestObject)
    self.data .+= 1
end

function reset!(self::TestObject)
    fill!(self.data, 0)
end

```

But now we get the "object.service()" pattern using `getproperty` to route names to specific functions.

```julia
function Base.getproperty(obj::TestObject, s::Symbol)
    if s === :increment
        # Return a closure that points to a pre-compiled top-level function
        return () -> _increment_logic(obj)
    elseif s === :reset
        return () -> _reset_logic(obj)
    else
        return getfield(obj, s)
    end
end

```

On using `obj.increment()`, Julia sees that `:increment` is a constant. Through constant propagation, it eliminates the `if/else` logic during compilation and replaces the call with the actual math, generating optimized code. At least in theory. To check this, try the example with `TestObject`,  and `@code_llvm increment!(a)` with `@code_llvm y1.f()`. And then compare `getproperty` to `@code_llvm a.increment!`


Comparing all of these approaches, `y1.f()` contains `@ijl_apply_generic`, 

```julia

using InteractiveUtils

call_stored(x) = x.f()
call_direct(x) = increment!(x)
call_property(x) = x.increment()

julia> @code_llvm call_stored(y1)  
; Function Signature: call_stored(Main.MockPythonMethod)
;  @ REPL[36]:1 within `call_stored`
define nonnull ptr @julia_call_stored_4309(ptr nocapture noundef nonnull readonly align 8 dereferenceable(16) %"x::MockPythonMethod") #0 {
top:
; ┌ @ Base_compiler.jl:54 within `getproperty`
   %"x::MockPythonMethod.f_ptr" = getelementptr inbounds i8, ptr %"x::MockPythonMethod", i64 8
   %"x::MockPythonMethod.f" = load atomic ptr, ptr %"x::MockPythonMethod.f_ptr" unordered, align 8
; └
  %0 = call nonnull ptr @ijl_apply_generic(ptr nonnull %"x::MockPythonMethod.f", ptr null, i32 0)
  ret ptr %0
}
```

While the other two approaches avoid it.


```julia
julia> @code_llvm call_direct(a)  
; Function Signature: call_direct(Main.TestObject)
;  @ REPL[37]:1 within `call_direct`
define nonnull ptr @julia_call_direct_4311(ptr nocapture noundef nonnull readonly align 8 dereferenceable(8) %"x::TestObject") #0 {
top:
  %0 = call nonnull ptr @"j_increment!_4313"(ptr nocapture nonnull readonly %"x::TestObject")
  ret ptr %0
}

julia> @code_llvm call_property(a)
; Function Signature: call_property(Main.TestObject)
;  @ REPL[38]:1 within `call_property`
define nonnull ptr @julia_call_property_4314(ptr nocapture noundef nonnull readonly align 8 dereferenceable(8) %"x::TestObject") #0 {
top:
  %0 = alloca [1 x [1 x ptr]], align 8
; ┌ @ REPL[8]:4 within `getproperty`
   %1 = load atomic ptr, ptr %"x::TestObject" unordered, align 8
; └
  store ptr %1, ptr %0, align 8
  %2 = call nonnull ptr @"j_#3_4316"(ptr nocapture nonnull readonly %0)
  ret ptr %2
}

```
