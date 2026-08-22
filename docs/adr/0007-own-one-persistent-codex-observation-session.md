# Own one persistent Codex observation session

The companion will own one long-lived independent `codex app-server --stdio` session and reuse it for quota and task observation instead of starting an app-server for every RPC or depending on the experimental global daemon. A transport failure invalidates and reaps the entire session process tree, then reconnects through a bounded circuit breaker; capability-level RPC failures continue to degrade independently under ADR-0006.

The dashboard's repeated app-server initialization is the controllable trigger for marketplace initialization storms, Codex is the producer and owner of leftover `marketplace-upgrade-*` staging data, and a large plugin such as ChatCut only amplifies the impact. The companion will therefore fix its session lifecycle without modifying ChatCut, managing Codex marketplace storage, or re-enabling its LaunchAgent before a real canary proves one app-server, zero staging growth, and no orphaned MCP processes.
