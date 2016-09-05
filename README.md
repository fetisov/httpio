# httpio
Cross Platform parser and response generator for the HTTP protocol.

Library has the simple API to embed to the custom systems.

Parser functions:
- Parses the HTTP stream on the fly
- Does not use the dynamic allocation (uses predefined connection buffer as the simple pool)
- Not used space of the connection buffer can be used as the network receiving buffer
- Supports the keep-alive multiple requests and various types of POST request

Generator functions:
- As well as the parser it does not use the dynamic allocation
- Organized by a ring buffer
- Supports the chunked transfer encoding

# Parser example (stdin/stdout instead the sockets)
`
`
