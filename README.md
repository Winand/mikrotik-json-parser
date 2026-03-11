# Mikrotik JSON Parser
Based on Chupakabra303's JSON parser for RouterOS.

ℹ️Consider using a **much** faster built-in `:deserialize` command.

⚠️As of hAP AC with RouterOS 7.21.3 both solutions are capable of parsing 64KB-1byte of data.
See also [#1](https://github.com/Winand/mikrotik-json-parser/issues/1).

Usage:
```
# Load library (set library functions to global variables)
/system script run "JParseFunctions"; global JSONLoad; global JSONLoads; global JSONUnload

# Parse data and print `ParsedResults[0].ParsedText` value
global content "{\"ParsedResults\": [{\"ParsedText\": \"Hello, world!\"}]}"
put ([$JSONLoads $content]->"ParsedResults"->0->"ParsedText")
set content

# or load JSON from file
put ([$JSONLoad "tmp"]->"ParsedResults"->0->"ParsedText")

# Unload library (clear global variables)
$JSONUnload
```

## Using built-in `:deserialize`
Since [RouterOS 7.13beta](https://forum.mikrotik.com/t/v7-13beta-testing-is-released/171132)
Mikrotik made this project obsolete by adding `:serialize` and `:deserialize` commands for converting values to/from JSON.
```
# Parse data and print `ParsedResults[0].ParsedText` value
global content "{\"ParsedResults\": [{\"ParsedText\": \"Hello, world!\"}]}"
put ([deserialize from=json value=$content]->"ParsedResults"->0->"ParsedText")
set content

# or load JSON from file
put ([deserialize from=json [/file/get "tmp" contents]]->"ParsedResults"->0->"ParsedText")
```

## Links
- [Embest: JSON Parser Mikrotik JParse](https://web.archive.org/web/20230530233616/http://www.embest.ru/mikrotik/json-parser-script)
- [Telegram бот для Mikrotik с Webhook и парсером JSON](https://habr.com/post/337978/)
