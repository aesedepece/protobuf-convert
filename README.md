Macros for convenient serialization of Rust data structures into/from Protocol Buffers.

# Introduction

This is a fork of [exonum-derive](https://crates.io/crates/exonum-derive) with
some changes to allow easier integration with other projects, and some new
features.

# Usage

First, add the dependency in `Cargo.toml`:

```
protobuf-convert = "0.1.0"
```

Then, define a `ProtobufConvert` trait:

```rust
trait ProtobufConvert {
    /// Type of the protobuf clone of Self 
    type ProtoStruct;

    /// Struct -> ProtoStruct
    fn to_pb(&self) -> Self::ProtoStruct;

    /// ProtoStruct -> Struct
    fn from_pb(pb: Self::ProtoStruct) -> Result<Self, Error>;
}
```

And to use it, import the trait and the macro:

For example, given the following protobuf:

```protobuf
message Ping {
    fixed64 nonce = 1;
}
```

rust-protobuf will generate the following struct:

```rust
#[derive(PartialEq,Clone,Default)]
#[cfg_attr(feature = "with-serde", derive(Serialize, Deserialize))]
pub struct Ping {
    // message fields
    pub nonce: u64,
    // special fields
    #[cfg_attr(feature = "with-serde", serde(skip))]
    pub unknown_fields: ::protobuf::UnknownFields,
    #[cfg_attr(feature = "with-serde", serde(skip))]
    pub cached_size: ::protobuf::CachedSize,
}
```

We may want to convert that struct into a more idiomatic one, and derive more traits.
This is the necessary code:

```rust
// Import trait
use crate::proto::ProtobufConvert;
// Import macro
use protobuf_convert::ProtobufConvert;
// Import module autogenerated by protocol buffers
use crate::proto::schema;

#[derive(ProtobofConvert)]
#[protobuf_convert(pb = "schema::Ping")]
struct Ping {
    nonce: u64,
}
```

Note that the `ProtobufConvert` trait must be implemented for all the fields,
see an example implementation for `u64`:

```rust
impl ProtobufConvert for u64 {
    type ProtoStruct = u64;
    fn to_pb(&self) -> Self::ProtoStruct {
	*self
    }
    fn from_pb(pb: Self::ProtoStruct) -> Result<Self, Error> {
	Ok(pb)
    }
}
```

Now, converting between `Ping` and `schema::Ping` can be done effortlessly.

A more complex example, featuring enums:

```protobuf
message Ping {
    fixed64 nonce = 1;
}
message Pong {
    fixed64 nonce = 1;
}
message Message {
    oneof kind {
        Ping Ping = 1;
        Pong Pong = 2;
    }
}
```

```rust
#[derive(ProtobofConvert)]
#[protobuf_convert(pb = "schema::Ping")]
struct Ping {
    nonce: u64,
}
#[derive(ProtobofConvert)]
#[protobuf_convert(pb = "schema::Pong")]
struct Pong {
    nonce: u64,
}
#[derive(ProtobofConvert)]
#[protobuf_convert(pb = "schema::Message")]
enum Message {
    Ping(Ping),
    Pong(Pong),
}
```

And it just works!

# See also

* [rust-protobuf](https://github.com/stepancheg/rust-protobuf)

