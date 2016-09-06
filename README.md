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

# Parser example (stdin instead the socket)
```c
#include <stdio.h>
#include "httpio.h"

static char in_pool[2048];

int main(int argc, const char **argv)
{
    FILE *f;
    httpi_t *in;
    f = argc == 2 ? fopen(argv[1], "rb") : stdin;
    in = httpi_init(in_pool, 2048);
    while (1)
    {
        int size;
        char *data;
        int status = httpi_pull(in);
        switch (status)
        {
        case HTTP_IN_NODATA:
            /* data input */
            httpi_get_state(in, &data, &size);
            size = fread(data, 1, size, f);
            httpi_push(in, data, size);
            if (size == 0) return 0;
            continue;
        case HTTP_IN_REQUEST:
            printf("page %s requested\n", httpi_request(in)->uri);
            continue;
        case HTTP_IN_POSTDATA:
            printf("new chunk of POST data \"%s\"\n", httpi_posted(in)->name);
            continue;
        case HTTP_IN_FINISHED:
            printf("request finished\n");
            continue;
        default:
            printf("invalid request!\n");
            return 0;
        }
    }
}
```
Example input:
```http
POST /cgi-bin/login.cgi HTTP/1.1
Host: www.tutorialspoint.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 35
Accept-Language: en-us
Accept-Encoding: gzip, deflate
Connection: Keep-Alive

login=Petya%20Vasechkin&password=qq
```
Example output:
```
page /cgi-bin/login.cgi requested
new chunk of POST data "login"
new chunk of POST data "password"
request finished
```
