# DebugTools AI for Claude Code

When the user asks to use DebugTools, inspect status, invoke Java methods, attach JVMs, generate DebugTools method parameters, list run configurations, start a run configuration with DebugTools Hotswap, reload code, inspect logs/SQL, or read Spring runtime configuration keys, follow `docs/workflow.md`.

Method invocation flow:

1. `get_debug_tools_status` when a complete target snapshot is useful, otherwise `list_debug_tools_connections`
2. `list_attachable_jvms` if no suitable connection exists
3. `attach_local_jvm`
4. `generate_method_args_template` for parameterized methods
5. `invoke_java_method` (use `resultView=JSON|DEBUG|NONE` only when requested)

After fresh attach or Hotswap startup, call direct DebugTools HTTP `GET /spring/ready` before invoking Spring-like Controller/Service/Bean methods. Use only the selected MCP connection `host` and `httpPort`; poll `STARTING` with `retryable=true`, stop on `retryable=false`, and do not guess localhost/default ports.

Preserve generated parameter names and method declaration order. Do not wrap `argsJson` in `targetMethodContent`.

Hotswap flow:

1. `list_debug_tools_run_configurations` if the run configuration name is unknown or ambiguous
2. `execute_debug_tools_run_configuration`
3. `compile_and_reload_modified_files` when recent Java changes need IDEA Java Debugger HotSwap reload
4. `get_hotswap_operation` when a reload returns an `operationId` or times out

Treat Hotswap `success=true` as "startup was requested", not as proof that DebugTools is connected.

Spring config flow:

1. Ask for config keys if the user did not provide any
2. `list_debug_tools_connections`
3. Attach with `list_attachable_jvms` and `attach_local_jvm` if no suitable connection exists
4. Use selected connection `host` and `httpPort` for `POST /spring/config` with a JSON string array body

Report returned values as Spring runtime Environment resolved values, not as direct file reads.

Code search flow:

1. `search_http_url` for HTTP routes, `search_symbol` for classes/methods, `search_text`/`search_regex`/`search_file` for other lookups
2. `get_file_problems` for IDE-reported problems in one file
3. Unknown parameters are silently ignored: use exact parameter names and inspect the raw response shape (`search_http_url` wraps results in `data.items`; the other search tools return `items` directly)

Database flow (read-only):

1. `list_database_connections`, then `list_database_schemas` with `connectionId`
2. `execute_sql_query` with empty `databaseName` and uppercase Oracle `schemaName`; keep queries bounded

Use `run_and_invoke` for the complete reload → invoke → optional logs/SQL loop. It may start only with an exact `runConfigurationName` and `allowStart=true`, or attach only with an explicit `pid` and `allowAttach=true`.
