# Yottapack 

Yottapack is a bandwidth and cpu efficient 

- Size efficient
- Efficient streaming
- Query fields
- Schema or schemaless


# Format

## Header
- Intro byte
- 1+ size byte (needed for streaming)
- (optional): Heads encoding

### Fat heads encoding

Not needed if schema or streaming. Encode strings
representing the name of fields, (and nested fields?)

- 1 type byte (is it needed?)
- 1+ byte size
- N+ chars for value

### Light heads encoding

If with schema

- 1+ byte size for each record
- 0 for some, if order is guaranteed

## Data

- 1 type byte (optional)
- 1+ size byte
- N+ value byte

# Special bytes

## Intro byte

1 byte for versioning and flags

## Forbidden sequences

- 0x00, 0x00, ETX
- 0x00, ETX, EOT


## Type byte

- 1 bit isArray
- 1 bit isMap
- 1 bit reserved
- 5 bit type (32 types)

### List of types

- Char
- Binary
- Hex
- Bool
- Nil
- Integers ( 2 * 4 )
- BigInt
- Floats (2)
- BigDecimal
- Time (2)
- Geolocation
- Dynamic
- Lookup
- Padding
- Reserved (9)
- Extended (2 bytes) 

Total: 32

## Length encoding

1+ byte of u8. Like utf8

1st byte signal if you should read also next byte.

- 0 -> 128: 1 byte
- 128+1 -> 2^14-1: 2 bytes (first byte is 1, 8th byte is 0)
- 2^14+1 -> 2^21-1: 3 bytes (first byte is 1, 8th byte is 1, 
16th byte is 1)
- and so on....

## Separators

EOT or ETX used as separators. Escaped in strings