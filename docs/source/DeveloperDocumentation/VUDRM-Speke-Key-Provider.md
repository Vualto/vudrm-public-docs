# SPEKE KEY PROVIDER API
These are the docs for the speke key provider, which has a speke endpoint for use with AWS. To learn how to use our SPEKE API with AWS' Media Services please read our [guide](/projects/vudrm/en/latest/DeveloperDocumentation/aws.html).

## Endpoints
Production url: https://speke.vudrm.tech/
Staging url: https://speke.staging.vudrm.tech/

## SPEKE
To retrieve drm information formatted to the AWS SPEKE standard send a POST request, formatting below, to **https://speke.vudrm.tech//client-name/speke**, where "client-name" is the name of the client. 

### Examples  
#### Requests
#### Headers
```
Content-Type: application/xml 
API-KEY: <api-key>
```
#### Widevine and Playready
```
<?xml version="1.0" encoding="UTF-8"?>
<cpix:CPIX id="someContentId" xmlns:cpix="urn:dashif:org:cpix" xmlns:pskc="urn:ietf:params:xml:ns:keyprov:pskc" xmlns:speke="urn:aws:amazon:com:speke">
   <cpix:ContentKeyList>
      <cpix:ContentKey kid="" explicitIV=""/>
   </cpix:ContentKeyList>
   <cpix:DRMSystemList>
      <!-- playready -->
      <cpix:DRMSystem kid="" systemId="9a04f079-9840-4286-ab92-e65be0885f95"> 
      </cpix:DRMSystem>
      <!-- widevine -->
      <cpix:DRMSystem kid="" systemId="edef8ba9-79d6-4ace-a3c8-27dcd51d21ed">
      </cpix:DRMSystem>
   </cpix:DRMSystemList>
</cpix:CPIX>
```
#### Fairplay
```
<?xml version="1.0" encoding="UTF-8"?>
<cpix:CPIX id="someContentId" xmlns:cpix="urn:dashif:org:cpix" xmlns:pskc="urn:ietf:params:xml:ns:keyprov:pskc" xmlns:speke="urn:aws:amazon:com:speke">
   <cpix:ContentKeyList>
      <cpix:ContentKey kid="" explicitIV=""/>
   </cpix:ContentKeyList>
   <cpix:DRMSystemList>
      <!-- fairplay -->
      <cpix:DRMSystem kid="" systemId="94ce86fb-07ff-4f43-adb8-93d2fa968ca2">
      </cpix:DRMSystem>
   </cpix:DRMSystemList>
</cpix:CPIX>
```

#### Responses
#### Headers
```
Content-Type: application/xml
```
#### Widevine and Playready
```
<?xml version="1.0" encoding="UTF-8"?>
<cpix:CPIX id="someContentId" xmlns:cpix="urn:dashif:org:cpix" xmlns:pskc="urn:ietf:params:xml:ns:keyprov:pskc" xmlns:speke="urn:aws:amazon:com:speke">
    <cpix:ContentKeyList>
        <cpix:ContentKey explicitIV="" kid="someKid">
            <cpix:Data>
                <pskc:Secret>
                    <pskc:PlainValue>playreadyContentKey</pskc:PlainValue>
                </pskc:Secret>
            </cpix:Data>
        </cpix:ContentKey>
    </cpix:ContentKeyList>
    <cpix:DRMSystemList>
        <cpix:DRMSystem kid="someKid" systemId="9a04f079-9840-4286-ab92-e65be0885f95">
            <speke:ProtectionHeader>playreadyProtectionHeader</speke:ProtectionHeader>
            <cpix:PSSH>playreadyPSSHBox</cpix:PSSH>
        </cpix:DRMSystem>
        <cpix:DRMSystem kid="someKid" systemId="edef8ba9-79d6-4ace-a3c8-27dcd51d21ed">
            <PSSH>widevinePSSHBox</PSSH>
        </cpix:DRMSystem>
    </cpix:DRMSystemList>
</cpix:CPIX>
```
#### Fairplay
```
<?xml version="1.0" encoding="UTF-8"?>
<cpix:CPIX id="someContentId" xmlns:cpix="urn:dashif:org:cpix" xmlns:pskc="urn:ietf:params:xml:ns:keyprov:pskc" xmlns:speke="urn:aws:amazon:com:speke">
    <cpix:ContentKeyList>
        <cpix:ContentKey explicitIV="fairplayIvHex-base64-encoded" kid="someKid">
            <cpix:Data>
                <pskc:Secret>
                    <pskc:PlainValue>fairplayKeyHex-base64-encoded</pskc:PlainValue>
                </pskc:Secret>
            </cpix:Data>
        </cpix:ContentKey>
    </cpix:ContentKeyList>
    <cpix:DRMSystemList>
        <cpix:DRMSystem kid="someKid" systemId="94ce86fb-07ff-4f43-adb8-93d2fa968ca2">
            <URIExtXKey>fairplayLaurl-base64-encoded</URIExtXKey>
            <KeyFormat>fairplayKeyFormat-base64-encoded</KeyFormat>
            <KeyFormatVersions>fairplayKeyFormatVersion-base64-encoded</KeyFormatVersions>
        </cpix:DRMSystem>
    </cpix:DRMSystemList>
</cpix:CPIX>
```
