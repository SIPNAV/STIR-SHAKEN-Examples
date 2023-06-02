how to setup asterisk to use Sipnav Stir Shaken API

Enable the following modules
`
load = func_logic.so
load = res_curl.so
load = func_curl.so
load = res_config_curl.so
load = func_json.so
`

Add the Following Dialplan


```
[sipnav-ss-sign]

    exten => s,1,NoOp(SipNav Shaken/Stir Sign Request Function)
     same => n,NoOp(Identity is returned in SS_IDENTITY or is empty on error)
     same => n,NoOp(Call like: n,GoSub(sipnav-ss-sign,s,1(${TO},${FROM},${ORIGID}${ATTEST_LVL}))
     same => n,Set(SS_APIKEY=YOUR_API_KEY_HERE)
     same => n,Set(SS_TO=${ARG1})
     same => n,Set(SS_FROM=${ARG2})
     same => n,Set(SS_ORIGID=${ARG3})
     same => n,Set(SS_ATTEST=${IF($["${ARG4}" = ""]?A:${ARG4})})
     same => n,NoOp(Build request data)
     same => n,Set(SS_DATA=api_key=${SS_APIKEY}&signingRequest[attest]=${SS_ATTEST}&signingRequest[dest][tn]=${SS_TO}&signingRequest[orig][tn]=${SS_FROM}&signingRequest[origid]=${SS_ORIGID}&signingRequest[iat]=${EPOCH})
     same => n,Set(__CURL_RESULT=${CURL(https://ssapi.sipnav.net/api/,${SS_DATA})})
     same => n,Set(__SS_IDENTITY=${JSON_DECODE(CURL_RESULT,signingResponse.identity)})
     same => n,Set(__SS_ERROR=${JSON_DECODE(CURL_RESULT,message)})
     same => n,Return()

```

Add testing Context and extensions


```
[stirtest]
 exten => 1,1,Answer()
  same => n,Wait(5)
  same => n,Hangup()

 exten => ss,1,Answer()
  same => n,GoSub(sipnav-ss-sign,s,1(17145555555,19495553535,78fabe9b-ef19-4b91-adaf-65f6e0a34e01,A))
  same => n,Verbose(${SS_IDENTITY})
  same => n,Verbose(${CURL_RESULT})
```


test manually from the shell on your asterisk box after ensuring config has been loaded. (Ensuring the configuration has been loaded is beyond the scope of this documentation. Restart Asterisk if unsure.)

```
asterisk -rx "channel originate Local/1@stirtest extension ss@stirtest"
```

use normal Asterisk tools such as ``` set(PJSIP_HEADER([add|update],Identity)=$SS_IDENTITY)``` to set the identity header in your dialplan. Contact support for your asterisk deployment for further assistance as depending on the base Asterisk Deployment (FreePBX etc) there may be additional ways to properly ensure this is set.



