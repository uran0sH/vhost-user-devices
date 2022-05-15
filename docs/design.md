# Design
the Common crate is used to process the communication with the vhost-use backend and also includes some other common features.

[vhost-user-net](./vhost-user-net.md)

[vhost-user-blk](./vhost-user-blk.md)

[vhost-user-fs](./vhost-user-fs.md)

# Common
There are two main modules in Common crate, one is Client and the other is Message. Master is responsible for communication with the backend and forwarding of messages. Message is the encapsulation of the communication protocol(vhost-user protocol).

## Master

```rust
struct Master {
    // Used to send requests to the vhost user backend in userspace.
    sock: VhostUserSock,
    // Maximum number of queues which is supported.
    max_queue_num: u64,
    // EventFd for client reset.
    delete_evt: EventFd,
}
```

## Message
[request header](https://qemu.readthedocs.io/en/latest/interop/vhost-user.html#header)
```rust
pub struct VhostUserMsgHdr {
    /// The request id for vhost-user message
    pub request: u32,
    /// The flags for property setting
    pub flags: u32,
    /// The total length of vhost user message
    pub size: u32,
}
```
header type
```rust
pub enum VhostUserHdrFlag {
    /// Bits[0..1] is message version number.
    Version = 0x3,
    /// Bits[2] Mark message as reply.
    Reply = 0x4,
    /// Bits[3] Sender anticipates a reply message from the peer.
    NeedReply = 0x8,
    /// All valid bits.
    AllFlags = 0xc,
    /// All reserved bits.
    ReservedBits = !0xf,
}
```

[Master message types](https://qemu.readthedocs.io/en/latest/interop/vhost-user.html#master-message-types):

```rust
pub enum VhostUserMsgReq {
    None = 0,
    GetFeatures = 1,
    SetFeatures = 2,
    SetOwner = 3,
    ResetOwner = 4,
    SetMemTable = 5,
    SetLogBase = 6,
    SetLogFd = 7,
    SetVringNum = 8,
    SetVringAddr = 9,
    SetVringBase = 10,
    GetVringBase = 11,
    SetVringKick = 12,
    SetVringCall = 13,
    SetVringErr = 14,
    GetProtocolFeatures = 15,
    SetProtocolFeatures = 16,
    GetQueueNum = 17,
    SetVringEnable = 18,
    SendRarp = 19,
    NetSetMtu = 20,
    SetSlaveReqFd = 21,
    IotlbMsg = 22,
    SetVringEndian = 23,
    GetConfig = 24,
    SetConfig = 25,
    CreateCryptoSession = 26,
    CloseCryptoSession = 27,
    PostcopyAdvise = 28,
    PostcopyListen = 29,
    PostcopyEnd = 30,
    GetInflightFd = 31,
    SetInflightFd = 32,
    MaxCmd = 33,
}
```
[A vring address description](https://qemu.readthedocs.io/en/latest/interop/vhost-user.html#a-vring-address-description)
```rust
pub struct VhostUserVringAddr {
    /// Index for virtual ring.
    pub index: u32,
    /// The option for virtual ring.
    pub flags: u32,
    /// Address of the descriptor table.
    pub desc_user_addr: u64,
    /// Address of the used ring.
    pub used_user_addr: u64,
    /// Address of the available ring.
    pub avail_user_addr: u64,
    /// Guest address for logging.
    pub log_guest_addr: u64,
}
```

[Memory regions description](https://qemu.readthedocs.io/en/latest/interop/vhost-user.html#memory-regions-description)
```rust
pub struct VhostUserMemHdr {
    /// Number of memory regions in the payload.
    pub nregions: u32,
    /// Padding for alignment.
    pub padding: u32,
}
```
A region is:
```rust
pub struct RegionMemInfo {
    /// Guest physical address of the memory region.
    pub guest_phys_addr: u64,
    /// Size of the memory region.
    pub memory_size: u64,
    /// Virtual address in the current process.
    pub userspace_addr: u64,
    /// Offset where region starts in the mapped memory.
    pub mmap_offset: u64,
}
```
An array of `RegionMemInfo`
```rust
pub struct VhostUserMemContext {
    /// The vector of memory region information.
    pub regions: Vec<RegionMemInfo>,
}
```
[Vring area description](https://qemu.readthedocs.io/en/latest/interop/vhost-user.html#vring-area-description)
```rust
pub struct VhostUserVringState {
    /// Index for virtual ring.
    pub index: u32,
    /// A common 32bit value to encapsulate vring state etc.
    pub value: u32,
}
```
