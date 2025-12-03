---
title: System calls
weight: 3
---

The firmware in TKey from the Castor release and onwards supports
system calls. You will typically use them through the system call
wrappers in [tkey-libs](https://github.com/tillitis/tkey-libs/)

## `sys_reset`

`int sys_reset(struct reset *rst, size_t len)`

Reset the TKey. Leave the reset type (`enum reset_start`) in `rst` as
well as an optional `app_digest`, forcing firmware to only allow that
specific app digest, as well as some data to leave to the next app in
the chain in `next_app_data`. Send the length of the `next_app_data`
in `len`.

Typically doesn't return. The TKey is reset and firmware starts again.

If it returns, it returns an error, probably because the reset length
being too large or that `rst` was pointing to memory outside of RAM.

## `sys_alloc`

`int sys_alloc(void)`

Allocate 128 kiB storage area for the current app. Must be done before
`sys_write()` or `sys_read()`. If the current app already has an area
allocated no new area will be allocated.

Returns 0 on success.

## `sys_dealloc`

`int sys_dealloc(void)`

Free an already allocated flash area for the current app.

Returns 0 on success.

## `sys_write`

`int sys_write(uint32_t offset, void *buf, size_t len)`

Write data in `buf` to the app's flash area at byte `offset` within
the area.

Up to storage area size bytes can be written at once and `offset` must
be a multiple of 256 bytes.

Returns 0 on success.

## `sys_read`

`int sys_read(uint32_t offset, void *buf, size_t len)`

Read `len` bytes into `buf` at byte `offset` from the app's flash
area.

Returns 0 on success.

## `sys_erase`

`int sys_erase(uint32_t offset, size_t len)`

Erase `len` bytes from `offset` within the area.

Both `len` and  `offset` must be a multiple of 4096 bytes.

Returns 0 on success.

## `sys_get_vidpid`

`int sys_get_vidpid(void)`

Returns the TKey Vendor and Product ID.

## `sys_preload_delete`

`int sys_preload_delete(void)`

Delete the app in flash slot 1. Only available for the verified
management app.

Returns 0 on success.

## `sys_preload_store`

`int sys_preload_store(uint32_t offset, void *app, size_t len)`

Store an app, or possibly just a block of an app, from the `app`
buffer in flash slot 1 at byte `offset`.

If you can't fit your entire app in the buffer, call
`sys_preload_store` many times as you receive the binary from the
client.

Up to preloaded app area size bytes can be written at once and
`offset` must be a multiple of 256 bytes.

Only available for the verified management app.

Returns 0 on success.

## `sys_preload_store_fin`

`int sys_preload_store_fin(size_t len, uint8_t digest[32], uint8_t signature[64])`

Finalize storing of an app where the complete binary size is `len`
in flash slot 1. Returns 0 on success. Only available for the
verified management app.

Compute a BLAKE2s hash digest over the entire binary. Pass the
result in `app_digest`.

Sign `app_digest` with your Ed25519 private key and pass the
resulting signature in `app_signature`.

Returns 0 on success.

## `sys_get_digsig`

`int sys_get_digsig(uint8_t digest[32], uint8_t signature[64])`

Copies the digest and signature of app in flash slot 1 to
`app_digest` and `app_signature`. Returns 0 on success. Only
available for the verified management app.

Returns 0 on success.

## `sys_status`

`int sys_status(void)`

Returns filesystem status. Non-zero when problems have been detected,
so far only that the first copy of the partition table didn't pass
checks.

## `sys_reset_data`

`void sys_reset_data(uint8_t next_app_data[RESET_DATA_SIZE])`

Fills in data left from previous app in the chain into
`next_app_data`. Buffer needs to be large enough to receive
RESET_DATA_SIZE bytes.
