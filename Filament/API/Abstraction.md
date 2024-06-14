# Abstraction

## Object

### Object Construction

Use pImpl.

#### Object Construction Interface

`VertexBuffer.h`

```Cpp
class UTILS_PUBLIC VertexBuffer : public FilamentAPI {
    struct BuilderDetails;

public:
    using AttributeType = backend::ElementType;
    using BufferDescriptor = backend::BufferDescriptor;

    class Builder : public BuilderBase<BuilderDetails> {
        friend struct BuilderDetails;
    public:
        Builder() noexcept;
        Builder& bufferCount(uint8_t bufferCount) noexcept;
        // ...
    }
}
```

`FilamentAPI` - `BuilderBase`

```Cpp
template<typename T>
using BuilderBase = utils::PrivateImplementation<T>;
```

`PrivateImplementation`

```Cpp
template<typename T>
class PrivateImplementation {
protected:
    T* mImpl = nullptr;
}
```

#### Object Construction Source

`VertexBuffer.cpp`

```Cpp
// set buffer count
VertexBuffer::Builder& VertexBuffer::Builder::bufferCount(uint8_t bufferCount) noexcept {
    mImpl->mBufferCount = bufferCount;
    return *this;
}

// end of the construction chain. Returns the built object.
VertexBuffer* VertexBuffer::Builder::build(Engine& engine) {
    // we'll examine downcast later
    return downcast(engine).createVertexBuffer(*this);
}
```

#### Object Construction Usage

`hellotriangle.cpp`

```Cpp
app.vb = VertexBuffer::Builder()
    .vertexCount(3)
    .bufferCount(1)
    .attribute(VertexAttribute::POSITION, 0, VertexBuffer::AttributeType::FLOAT2, 0, 12)
    .attribute(VertexAttribute::COLOR, 0, VertexBuffer::AttributeType::UBYTE4, 8, 12)
    .normalized(VertexAttribute::COLOR)
    .build(*engine);
```

#### Analysis of Object Construction

Why pimpl?

### Class Method

Use downcast.

#### Class Method Interface

`VertexBuffer.h`

```Cpp
size_t VertexBuffer::getVertexCount() const noexcept;
```

#### Class Method Source

`VertexBuffer.cpp`

- Use a macro to register the downcast function for this class
- It static_casts an `Interface` class to the `FInterface`, which is the actual implementation

```Cpp
size_t VertexBuffer::getVertexCount() const noexcept {
    return downcast(this)->getVertexCount();
}
```

`details/VertexBuffer`

`FInterface` typically directly refers to other `FInterfaces`.

```Cpp
class FBufferObject;
class FEngine;

class FVertexBuffer : public VertexBuffer {
public:
    size_t getVertexCount() const noexcept;
private:
    friend class VertexBuffer;
};

FILAMENT_DOWNCAST(VertexBuffer) // registers downcast in the FInterface header

// implementation in FInterface source
size_t FVertexBuffer::getVertexCount() const noexcept {
    return mVertexCount;
}
```

#### Analysis of Class Method

Why downcast pattern? Why not pimpl, or even virtual function?