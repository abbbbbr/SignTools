# How does this all work?

This project is not one simple program. It is a combination of a web service and a builder, which work together to achieve signing and sideloading.

Below is a rough [sequence diagram](https://en.wikipedia.org/wiki/Sequence_diagram) of how the entire process works. If you haven't read a diagram like this before, it essentialy describes interactions between different parties. In this case, we have four parties: the User, Web Service, Builder, and Apple. Start reading the diagram from the top and make your way to the bottom. Each vertical line is a party, while each horizontal line is an interaction. The big rectangle labeled `alt - if using a developer account` will only be executed if you are signing with a developer account. Otherwise, it is skipped.

[![](https://mermaid.ink/img/eyJjb2RlIjoic2VxdWVuY2VEaWFncmFtXG4gICAgVXNlciAtPj5XZWIgU2VydmljZTogVXBsb2FkIHVuc2lnbmVkIGFwcFxuICAgIFdlYiBTZXJ2aWNlLT4-V2ViIFNlcnZpY2U6IFNhdmUgYXBwIGFuZCBnZW5lcmF0ZSBzaWduIGpvYlxuICAgIFdlYiBTZXJ2aWNlLT4-QnVpbGRlcjogVHJpZ2dlciAoYWN0aXZhdGUpXG4gICAgICAgIEJ1aWxkZXItPj5XZWIgU2VydmljZTogUmV0cmlldmUgbGFzdCBzaWduIGpvYlxuICAgICAgICBXZWIgU2VydmljZS0-PkJ1aWxkZXI6IFxuICAgICAgICBub3RlIG92ZXIgQnVpbGRlcjogVGhlIHNpZ24gam9iIGlzIGFuIGFyY2hpdmUgb2YgPGJyPiBmaWxlcyBzdWNoIGFzIHRoZSBzaWduaW5nIGNlcnRpZmljYXRlLCA8YnI-IGRldmVsb3BlciBhY2NvdW50IChpZiB1c2VkKSwgPGJyPiBhbmQgdW5zaWduZWQgYXBwXG4gICAgYWx0IGlmIHVzaW5nIGEgZGV2ZWxvcGVyIGFjY291bnRcbiAgICByZWN0IHJnYigwLCAwLCAyNTUsIC4xKVxuICAgICAgICBXZWIgU2VydmljZS0-PlVzZXI6IE5hdmlnYXRlIHRvIDJGQSBwYWdlXG4gICAgICAgIG5vdGUgb3ZlciBXZWIgU2VydmljZTogMkZBID0gVHdvLWZhY3RvciBhdXRoZW50aWNhdGlvblxuICAgICAgICBCdWlsZGVyIC0-PkFwcGxlOiBTdGFydCBsb2cgaW4gdG8gYWNjb3VudFxuICAgICAgICBBcHBsZS0-PlVzZXI6IFNlbmQgMkZBIGNvZGVcbiAgICAgICAgVXNlci0-PldlYiBTZXJ2aWNlOiBTdWJtaXQgMkZBIGNvZGVcbiAgICAgICAgQnVpbGRlci0-PldlYiBTZXJ2aWNlOiBSZXRyaWV2ZSAyRkEgY29kZVxuICAgICAgICBXZWIgU2VydmljZS0-PkJ1aWxkZXI6IFxuICAgICAgICBCdWlsZGVyLT4-QXBwbGU6IEZpbmlzaCBsb2cgaW4gdG8gYWNjb3VudFxuICAgIGVuZFxuICAgIGVuZFxuICAgIFdlYiBTZXJ2aWNlLT4-VXNlcjogTmF2aWdhdGUgdG8gZGFzaGJvYXJkXG4gICAgQnVpbGRlciAtPj5CdWlsZGVyOiBTaWduIHRoZSBhcHBcbiAgICBCdWlsZGVyIC0-PldlYiBTZXJ2aWNlOiBVcGxvYWQgc2lnbmVkIGFwcFxuICAgIFVzZXItPj5XZWIgU2VydmljZTogSW5zdGFsbCBzaWduZWQgYXBwXG4gICAgV2ViIFNlcnZpY2UtPj5Vc2VyOiBcbiAgICBVc2VyLT4-VXNlcjogRG9uZSIsIm1lcm1haWQiOnt9LCJ1cGRhdGVFZGl0b3IiOmZhbHNlfQ)](https://mermaid-js.github.io/mermaid-live-editor/#/edit/eyJjb2RlIjoic2VxdWVuY2VEaWFncmFtXG4gICAgVXNlciAtPj5XZWIgU2VydmljZTogVXBsb2FkIHVuc2lnbmVkIGFwcFxuICAgIFdlYiBTZXJ2aWNlLT4-V2ViIFNlcnZpY2U6IFNhdmUgYXBwIGFuZCBnZW5lcmF0ZSBzaWduIGpvYlxuICAgIFdlYiBTZXJ2aWNlLT4-QnVpbGRlcjogVHJpZ2dlciAoYWN0aXZhdGUpXG4gICAgICAgIEJ1aWxkZXItPj5XZWIgU2VydmljZTogUmV0cmlldmUgbGFzdCBzaWduIGpvYlxuICAgICAgICBXZWIgU2VydmljZS0-PkJ1aWxkZXI6IFxuICAgICAgICBub3RlIG92ZXIgQnVpbGRlcjogVGhlIHNpZ24gam9iIGlzIGFuIGFyY2hpdmUgb2YgPGJyPiBmaWxlcyBzdWNoIGFzIHRoZSBzaWduaW5nIGNlcnRpZmljYXRlLCA8YnI-IGRldmVsb3BlciBhY2NvdW50IChpZiB1c2VkKSwgPGJyPiBhbmQgdW5zaWduZWQgYXBwXG4gICAgYWx0IGlmIHVzaW5nIGEgZGV2ZWxvcGVyIGFjY291bnRcbiAgICByZWN0IHJnYigwLCAwLCAyNTUsIC4xKVxuICAgICAgICBXZWIgU2VydmljZS0-PlVzZXI6IE5hdmlnYXRlIHRvIDJGQSBwYWdlXG4gICAgICAgIG5vdGUgb3ZlciBXZWIgU2VydmljZTogMkZBID0gVHdvLWZhY3RvciBhdXRoZW50aWNhdGlvblxuICAgICAgICBCdWlsZGVyIC0-PkFwcGxlOiBTdGFydCBsb2cgaW4gdG8gYWNjb3VudFxuICAgICAgICBBcHBsZS0-PlVzZXI6IFNlbmQgMkZBIGNvZGVcbiAgICAgICAgVXNlci0-PldlYiBTZXJ2aWNlOiBTdWJtaXQgMkZBIGNvZGVcbiAgICAgICAgQnVpbGRlci0-PldlYiBTZXJ2aWNlOiBSZXRyaWV2ZSAyRkEgY29kZVxuICAgICAgICBXZWIgU2VydmljZS0-PkJ1aWxkZXI6IFxuICAgICAgICBCdWlsZGVyLT4-QXBwbGU6IEZpbmlzaCBsb2cgaW4gdG8gYWNjb3VudFxuICAgIGVuZFxuICAgIGVuZFxuICAgIFdlYiBTZXJ2aWNlLT4-VXNlcjogTmF2aWdhdGUgdG8gZGFzaGJvYXJkXG4gICAgQnVpbGRlciAtPj5CdWlsZGVyOiBTaWduIHRoZSBhcHBcbiAgICBCdWlsZGVyIC0-PldlYiBTZXJ2aWNlOiBVcGxvYWQgc2lnbmVkIGFwcFxuICAgIFVzZXItPj5XZWIgU2VydmljZTogSW5zdGFsbCBzaWduZWQgYXBwXG4gICAgV2ViIFNlcnZpY2UtPj5Vc2VyOiBcbiAgICBVc2VyLT4-VXNlcjogRG9uZSIsIm1lcm1haWQiOnt9LCJ1cGRhdGVFZGl0b3IiOmZhbHNlfQ)