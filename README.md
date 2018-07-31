# Mikrotik JSON Parser
Based on Chupakabra303's JSON parser for RouterOS.

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

See links below:

http://www.embest.ru/mikrotik/json-parser-script

https://habr.com/post/337978/
