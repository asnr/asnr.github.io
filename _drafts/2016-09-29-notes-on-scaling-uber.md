
Based on goto; conference talk "What I wish I knew before scaling Uber" by Matt Ranney https://www.youtube.com/watch?v=kb-m2fasdDY.

In particular I'm interested in the complexities particular to microservices architectures and not just general scaling problems.

- "servers are not browsers" - HTTP/REST/JSON is the wrong choice for RPC calls between microservices in a data centre
  * HTTP includes a lot of semantics (caching, choice of splitting of data/metadata between HTTP verbs, statuses, headers, query strings, request bodies) that make sense for a browser but complicate microservice communication
  * JSON is not typed. This works fine to begin with, but you'll pay for it later
- different languages
  * moving between teams
  * profiling looks different for different languages
  * Uber building tech to make profiling look the same (e.g. same flame graphs)
- metrics
  * teams building their own dashboards makes tracing problems through multiple services hard
  * Uber building consistent metrics tool that automatically measures new services with minimal dev input
- logging
  * needs to be consistent across services, languages
  * logging when problems hit can make the problem worse, Uber is now adding backpressure so logs get dropped when they can't be written quickly enough
- Fanout problems - hit worst case perf of at least one service w/ high prob. when talking to lots of services
  * to diagnose need some kind of tracing, even if just passing an initial request ID through all service calls (incidentally, this can be hard, esp. across multiple languages)
