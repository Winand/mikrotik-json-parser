# -------------------------------- JParseFunctions ---------------------------------------------------
# ------------------------------- fJParsePrint ----------------------------------------------------------------
:global fJParsePrint
:if (!any $fJParsePrint) do={ :global fJParsePrint do={
  :global JParseOut
  :local TempPath
  :global fJParsePrint

  :if ([:len $1] = 0) do={
    :set $1 "\$JParseOut"
    :set $2 $JParseOut
   }
   
  :foreach k,v in=$2 do={
    :if ([:typeof $k] = "str") do={
      :set k "\"$k\""
    }
    :set TempPath ($1. "->" . $k)
    :if ([:typeof $v] = "array") do={
      :if ([:len $v] > 0) do={
        $fJParsePrint $TempPath $v
      } else={
        :put "$TempPath = [] ($[:typeof $v])"
      }
    } else={
        :put "$TempPath = $v ($[:typeof $v])"
    }
  }
}}
# ------------------------------- fJParsePrintVar ----------------------------------------------------------------
:global fJParsePrintVar
:if (!any $fJParsePrintVar) do={ :global fJParsePrintVar do={
  :global JParseOut
  :local TempPath
  :global fJParsePrintVar
  :local fJParsePrintRet ""

  :if ([:len $1] = 0) do={
    :set $1 "\$JParseOut"
    :set $2 $JParseOut
   }
   
  :foreach k,v in=$2 do={
    :if ([:typeof $k] = "str") do={
      :set k "\"$k\""
    }
    :set TempPath ($1. "->" . $k)
    :if ($fJParsePrintRet != "") do={
      :set fJParsePrintRet ($fJParsePrintRet . "\r\n")
    }    
    :if ([:typeof $v] = "array") do={
      :if ([:len $v] > 0) do={
        :set fJParsePrintRet ($fJParsePrintRet . [$fJParsePrintVar $TempPath $v])
      } else={
        :set fJParsePrintRet ($fJParsePrintRet . "$TempPath = [] ($[:typeof $v])")
      }
    } else={
        :set fJParsePrintRet ($fJParsePrintRet . "$TempPath = $v ($[:typeof $v])")
    }
  }
  :return $fJParsePrintRet
}}
# ------------------------------- fJSkipWhitespace ----------------------------------------------------------------
:global fJSkipWhitespace
:if (!any $fJSkipWhitespace) do={ :global fJSkipWhitespace do={
  :global Jpos
  :global JSONIn
  :global Jdebug
  :while ($Jpos < [:len $JSONIn] and ([:pick $JSONIn $Jpos] ~ "[ \r\n\t]")) do={
    :set Jpos ($Jpos + 1)
  }
  :if ($Jdebug) do={:put "fJSkipWhitespace: Jpos=$Jpos Char=$[:pick $JSONIn $Jpos]"}
}}
# -------------------------------- fJParse ---------------------------------------------------------------
:global fJParse
:if (!any $fJParse) do={ :global fJParse do={
  :global Jpos
  :global JSONIn
  :global Jdebug
  :global fJSkipWhitespace
  :local Char

  :if (!$1) do={
    :set Jpos 0
   }
  
  $fJSkipWhitespace
  :set Char [:pick $JSONIn $Jpos]
  :if ($Jdebug) do={:put "fJParse: Jpos=$Jpos Char=$Char"}
  :if ($Char="{") do={
    :set Jpos ($Jpos + 1)
    :global fJParseObject
    :return [$fJParseObject]
  } else={
    :if ($Char="[") do={
      :set Jpos ($Jpos + 1)
      :global fJParseArray
      :return [$fJParseArray]
    } else={
      :if ($Char="\"") do={
        :set Jpos ($Jpos + 1)
        :global fJParseString
        :return [$fJParseString]
      } else={
#        :if ([:pick $JSONIn $Jpos ($Jpos+2)]~"^-\?[0-9]") do={
        :if ($Char~"[eE0-9.+-]") do={
          :global fJParseNumber
          :return [$fJParseNumber]
        } else={

          :if ($Char="n" and [:pick $JSONIn $Jpos ($Jpos+4)]="null") do={
            :set Jpos ($Jpos + 4)
            :return []
          } else={
            :if ($Char="t" and [:pick $JSONIn $Jpos ($Jpos+4)]="true") do={
              :set Jpos ($Jpos + 4)
              :return true
            } else={
              :if ($Char="f" and [:pick $JSONIn $Jpos ($Jpos+5)]="false") do={
                :set Jpos ($Jpos + 5)
                :return false
              } else={
                :put "Err.Raise 8732. No JSON object could be fJParseed"
                :set Jpos ($Jpos + 1)
                :return []
              }
            }
          }
        }
      }
    }
  }
}}

#-------------------------------- fJParseString ---------------------------------------------------------------
:global fJParseString
:if (!any $fJParseString) do={ :global fJParseString do={
  :global Jpos
  :global JSONIn
  :global Jdebug
  :global fUnicodeToUTF8
  :local Char
  :local StartIdx
  :local Char2
  :local TempString ""
  :local UTFCode
  :local Unicode

  :set StartIdx $Jpos
  :set Char [:pick $JSONIn $Jpos]
  :if ($Jdebug) do={:put "fJParseString: Jpos=$Jpos Char=$Char"}
  :while ($Jpos < [:len $JSONIn] and $Char != "\"") do={
    :if ($Char="\\") do={
      :set Char2 [:pick $JSONIn ($Jpos + 1)]
      :if ($Char2 = "u") do={
        :set UTFCode [:tonum "0x$[:pick $JSONIn ($Jpos+2) ($Jpos+6)]"]
        :if ($UTFCode>=0xD800 and $UTFCode<=0xDFFF) do={
# Surrogate pair
          :set Unicode  (($UTFCode & 0x3FF) << 10)
          :set UTFCode [:tonum "0x$[:pick $JSONIn ($Jpos+8) ($Jpos+12)]"]
          :set Unicode ($Unicode | ($UTFCode & 0x3FF) | 0x10000)
          :set TempString ($TempString . [:pick $JSONIn $StartIdx $Jpos] . [$fUnicodeToUTF8 $Unicode])         
          :set Jpos ($Jpos + 12)
        } else= {
# Basic Multilingual Plane (BMP)
          :set Unicode $UTFCode
          :set TempString ($TempString . [:pick $JSONIn $StartIdx $Jpos] . [$fUnicodeToUTF8 $Unicode])
          :set Jpos ($Jpos + 6)
        }
        :set StartIdx $Jpos
        :if ($Jdebug) do={:put "fJParseString Unicode: $Unicode"}
      } else={
        :if ($Char2 ~ "[\\bfnrt\"]") do={
          :if ($Jdebug) do={:put "fJParseString escape: Char+Char2 $Char$Char2"}
          :set TempString ($TempString . [:pick $JSONIn $StartIdx $Jpos] . [[:parse "(\"\\$Char2\")"]])
          :set Jpos ($Jpos + 2)
          :set StartIdx $Jpos
        } else={
          :if ($Char2 = "/") do={
            :if ($Jdebug) do={:put "fJParseString /: Char+Char2 $Char$Char2"}
            :set TempString ($TempString . [:pick $JSONIn $StartIdx $Jpos] . "/")
            :set Jpos ($Jpos + 2)
            :set StartIdx $Jpos
          } else={
            :put "Err.Raise 8732. Invalid escape"
            :set Jpos ($Jpos + 2)
          }
        }
      }
    } else={
      :set Jpos ($Jpos + 1)
    }
    :set Char [:pick $JSONIn $Jpos]
  }
  :set TempString ($TempString . [:pick $JSONIn $StartIdx $Jpos])
  :set Jpos ($Jpos + 1)
  :if ($Jdebug) do={:put "fJParseString: $TempString"}
  :return $TempString
}}

#-------------------------------- fJParseNumber ---------------------------------------------------------------
:global fJParseNumber
:if (!any $fJParseNumber) do={ :global fJParseNumber do={
  :global Jpos
  :local StartIdx
  :global JSONIn
  :global Jdebug
  :local NumberString
  :local Number

  :set StartIdx $Jpos   
  :set Jpos ($Jpos + 1)
  :while ($Jpos < [:len $JSONIn] and [:pick $JSONIn $Jpos]~"[eE0-9.+-]") do={
    :set Jpos ($Jpos + 1)
  }
  :set NumberString [:pick $JSONIn $StartIdx $Jpos]
  :set Number [:tonum $NumberString] 
  :if ([:typeof $Number] = "num") do={
    :if ($Jdebug) do={:put "fJParseNumber: StartIdx=$StartIdx Jpos=$Jpos $Number ($[:typeof $Number])"}
    :return $Number
  } else={
    :if ($Jdebug) do={:put "fJParseNumber: StartIdx=$StartIdx Jpos=$Jpos $NumberString ($[:typeof $NumberString])"}
    :return $NumberString
  }
}}

#-------------------------------- fJParseArray ---------------------------------------------------------------
:global fJParseArray
:if (!any $fJParseArray) do={ :global fJParseArray do={
  :global Jpos
  :global JSONIn
  :global Jdebug
  :global fJParse
  :global fJSkipWhitespace
  :local Value
  :local ParseArrayRet [:toarray ""]
  
  $fJSkipWhitespace    
  :while ($Jpos < [:len $JSONIn] and [:pick $JSONIn $Jpos]!= "]") do={
    :set Value [$fJParse true]
    :set ($ParseArrayRet->([:len $ParseArrayRet])) $Value
    :if ($Jdebug) do={:put "fJParseArray: Value="; :put $Value}
    $fJSkipWhitespace
    :if ([:pick $JSONIn $Jpos] = ",") do={
      :set Jpos ($Jpos + 1)
      $fJSkipWhitespace
    }
  }
  :set Jpos ($Jpos + 1)
#  :if ($Jdebug) do={:put "ParseArrayRet: "; :put $ParseArrayRet}
  :return $ParseArrayRet
}}

# -------------------------------- fJParseObject ---------------------------------------------------------------
:global fJParseObject
:if (!any $fJParseObject) do={ :global fJParseObject do={
  :global Jpos
  :global JSONIn
  :global Jdebug
  :global fJSkipWhitespace
  :global fJParseString
  :global fJParse
# Syntax :local ParseObjectRet ({}) don't work in recursive call, use [:toarray ""] for empty array!!!
  :local ParseObjectRet [:toarray ""]
  :local Key
  :local Value
  :local ExitDo false
  
  $fJSkipWhitespace
  :while ($Jpos < [:len $JSONIn] and [:pick $JSONIn $Jpos]!="}" and !$ExitDo) do={
    :if ([:pick $JSONIn $Jpos]!="\"") do={
      :put "Err.Raise 8732. Expecting property name"
      :set ExitDo true
    } else={
      :set Jpos ($Jpos + 1)
      :set Key [$fJParseString]
      $fJSkipWhitespace
      :if ([:pick $JSONIn $Jpos] != ":") do={
        :put "Err.Raise 8732. Expecting : delimiter"
        :set ExitDo true
      } else={
        :set Jpos ($Jpos + 1)
        :set Value [$fJParse true]
        :set ($ParseObjectRet->$Key) $Value
        :if ($Jdebug) do={:put "fJParseObject: Key=$Key Value="; :put $Value}
        $fJSkipWhitespace
        :if ([:pick $JSONIn $Jpos]=",") do={
          :set Jpos ($Jpos + 1)
          $fJSkipWhitespace
        }
      }
    }
  }
  :set Jpos ($Jpos + 1)
#  :if ($Jdebug) do={:put "ParseObjectRet: "; :put $ParseObjectRet}
  :return $ParseObjectRet
}}

# ------------------- fByteToEscapeChar ----------------------
:global fByteToEscapeChar
:if (!any $fByteToEscapeChar) do={ :global fByteToEscapeChar do={
#  :set $1 [:tonum $1]
  :return [[:parse "(\"\\$[:pick "0123456789ABCDEF" (($1 >> 4) & 0xF)]$[:pick "0123456789ABCDEF" ($1 & 0xF)]\")"]]
}}

# ------------------- fUnicodeToUTF8----------------------
:global fUnicodeToUTF8
:if (!any $fUnicodeToUTF8) do={ :global fUnicodeToUTF8 do={
  :global fByteToEscapeChar
#  :local Ubytes [:tonum $1]
  :local Nbyte
  :local EscapeStr ""

  :if ($1 < 0x80) do={
    :set EscapeStr [$fByteToEscapeChar $1]
  } else={
    :if ($1 < 0x800) do={
      :set Nbyte 2
    } else={  
      :if ($1 < 0x10000) do={
        :set Nbyte 3
      } else={
        :if ($1 < 0x20000) do={
          :set Nbyte 4
        } else={
          :if ($1 < 0x4000000) do={
            :set Nbyte 5
          } else={
            :if ($1 < 0x80000000) do={
              :set Nbyte 6
            }
          }
        }
      }
    }
    :for i from=2 to=$Nbyte do={
      :set EscapeStr ([$fByteToEscapeChar ($1 & 0x3F | 0x80)] . $EscapeStr)
      :set $1 ($1 >> 6)
    }
    :set EscapeStr ([$fByteToEscapeChar (((0xFF00 >> $Nbyte) & 0xFF) | $1)] . $EscapeStr)
  }
  :return $EscapeStr
}}

# ------------------- Load JSON from arg --------------------------------
global JSONLoads
if (!any $JSONLoads) do={ global JSONLoads do={
    global JSONIn $1
    global fJParse
    local ret [$fJParse]
    set JSONIn
    global Jpos; set Jpos
    global Jdebug; if (!$Jdebug) do={set Jdebug}
    return $ret
}}

# ------------------- Load JSON from file --------------------------------
global JSONLoad
if (!any $JSONLoad) do={ global JSONLoad do={
    if ([len [/file find name=$1]] > 0) do={
        global JSONLoads
        return [$JSONLoads [/file get $1 contents]]
    }
}}

# ------------------- Unload JSON parser library ----------------------
global JSONUnload
if (!any $JSONUnload) do={ global JSONUnload do={
    global JSONIn; set JSONIn
    global Jpos; set Jpos
    global Jdebug; set Jdebug
    global fByteToEscapeChar; set fByteToEscapeChar
    global fJParse; set fJParse
    global fJParseArray; set fJParseArray
    global fJParseNumber; set fJParseNumber
    global fJParseObject; set fJParseObject
    global fJParsePrint; set fJParsePrint
    global fJParsePrintVar; set fJParsePrintVar
    global fJParseString; set fJParseString
    global fJSkipWhitespace; set fJSkipWhitespace
    global fUnicodeToUTF8; set fUnicodeToUTF8
    global JSONLoads; set JSONLoads
    global JSONLoad; set JSONLoad
    global JSONUnload; set JSONUnload
}}
# ------------------- End JParseFunctions----------------------
