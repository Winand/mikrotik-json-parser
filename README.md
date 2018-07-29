# Mikrotik JSON Parser
Based on Chupakabra303's JSON parser for RouterOS.

Usage:
```
# Load library
/system script run "JParseFunctions"; :global JSONLoads; :global JSONUnload

# Parse data from `tmp` file and print ParsedResults[0].ParsedText value
:put ([$JSONLoads [/file get tmp contents]]->"ParsedResults"->0->"ParsedText")

# Unload library
$JSONUnload
```

See links below:

http://www.embest.ru/mikrotik/json-parser-script

https://habr.com/post/337978/
