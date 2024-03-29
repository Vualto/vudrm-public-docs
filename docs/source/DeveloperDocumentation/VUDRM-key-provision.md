# ENCRYPTION KEY PROVISION

The VUDRM key provision service exposes the generation of DRM encryption. It provides two secure REST APIs allowing you to retrieve encryption keys for all DRM types in order to encrypt your content.

## CPIX Key Provider API

We recommend using our CPIX API to fetch VUDRM encryption keys in CPIX XML document format. Compatible with Unified Streaming (version 1.9.3 and above).
This API can also be used to return the keys from the CPIX document in JSON format; for more information on this please see the [JSON keys](#json-keys) section.
All requests made to this API will require your API key set as the header `API-KEY`. If you do not know or have not been given your API key please contact support@vualto.com.

> Replace `<api-key>`, `<client-name>`, and `<content-id>` with appropiate values.

> `<content-id>` may only contain alphanumeric characters, underscores, and hyphens.

> Available DRM systems [`fairplay`, `playready`, `widevine`].

### Endpoints

#### Default

Get a CPIX 2.1 document with all DRM systems `<client-name>` is entitled to use.

##### Optional query params:

* `drm` : comma separated list of DRM systems to be included in the response.

##### Request

```bash
curl -XGET -H "API-KEY: <api-key>" \
"https://cpix.vudrm.tech/v1/cpix/<client-name>/<content-id>"
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<cpix:CPIX xmlns:pskc="urn:ietf:params:xml:ns:keyprov:pskc" xmlns:xsi="urn:ietf:params:xml:ns:keyprov:pskc" xmlns:cpix="urn:dashif:org:cpix" xsi:schemaLocation="urn:dashif:org:cpix cpix.xsd">
  <cpix:ContentKeyList>
    <cpix:ContentKey kid="11111111-1111-1111-1111-111111111111" explicitIV="YmFzZTY0ZW5jb2RlZAo=">
      <cpix:Data>
        <pskc:Secret>
          <pskc:PlainValue>YmFzZTY0ZW5jb2RlZAo=</pskc:PlainValue>
        </pskc:Secret>
      </cpix:Data>
    </cpix:ContentKey>
  </cpix:ContentKeyList>
  <cpix:DRMSystemList>
    <cpix:DRMSystem kid="11111111-1111-1111-1111-111111111111" systemId="9a04f079-9840-4286-ab92-e65be0885f95">
      <cpix:PSSH>YmFzZTY0ZW5jb2RlZAo=</cpix:PSSH>
    </cpix:DRMSystem>
    <cpix:DRMSystem kid="11111111-1111-1111-1111-111111111111" systemId="edef8ba9-79d6-4ace-a3c8-27dcd51d21ed">
      <cpix:PSSH>YmFzZTY0ZW5jb2RlZAo=</cpix:PSSH>
    </cpix:DRMSystem>
    <cpix:DRMSystem kid="11111111-1111-1111-1111-111111111111" systemId="94ce86fb-07ff-4f43-adb8-93d2fa968ca2">
      <cpix:URIExtXKey>YmFzZTY0ZW5jb2RlZAo=</cpix:URIExtXKey>
      <cpix:HLSSignalingData>YmFzZTY0ZW5jb2RlZAo=</cpix:HLSSignalingData>
    </cpix:DRMSystem>
  </cpix:DRMSystemList>
</cpix:CPIX>
```

#### Decrypt

Get a CPIX 2.1 document with only a single content key.

##### Optional query params:

* `drm` : comma separated list of DRM systems to be included in the response.

##### Request

```bash
curl -XGET -H "API-KEY: <api-key>" \
"https://cpix.vudrm.tech/v1/cpix/<client-name>/<content-id>/decrypt"
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<cpix:CPIX xmlns:pskc="urn:ietf:params:xml:ns:keyprov:pskc" xmlns:xsi="urn:ietf:params:xml:ns:keyprov:pskc" xmlns:cpix="urn:dashif:org:cpix" xsi:schemaLocation="urn:dashif:org:cpix cpix.xsd">
  <cpix:ContentKeyList>
    <cpix:ContentKey kid="11111111-1111-1111-1111-111111111111" explicitIV="YmFzZTY0ZW5jb2RlZAo=">
      <cpix:Data>
        <pskc:Secret>
          <pskc:PlainValue>YmFzZTY0ZW5jb2RlZAo=</pskc:PlainValue>
        </pskc:Secret>
      </cpix:Data>
    </cpix:ContentKey>
  </cpix:ContentKeyList>
</cpix:CPIX>
```

#### Multikey

Get a CPIX 2.1 document that uses separate keys for video and audio tracks.

##### Optional query params:

* `drm` : comma separated list of DRM systems to be included in the response.

##### Request

```bash
curl -XGET -H "API-KEY: <api-key>" \
"https://cpix.vudrm.tech/v1/cpix/<client-name>/<content-id>/multikey"
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<cpix:CPIX xmlns:pskc="urn:ietf:params:xml:ns:keyprov:pskc" xmlns:xsi="urn:ietf:params:xml:ns:keyprov:pskc" xmlns:cpix="urn:dashif:org:cpix" xsi:schemaLocation="urn:dashif:org:cpix cpix.xsd">
  <cpix:ContentKeyList>
    <cpix:ContentKey kid="11111111-1111-1111-1111-111111111111" explicitIV="YmFzZTY0ZW5jb2RlZAo=">
      <cpix:Data>
        <pskc:Secret>
          <pskc:PlainValue>YmFzZTY0ZW5jb2RlZAo=</pskc:PlainValue>
        </pskc:Secret>
      </cpix:Data>
    </cpix:ContentKey>
    <cpix:ContentKey kid="22222222-2222-2222-22222-22222222222" explicitIV="YmFzZTY0ZW5jb2RlZAo=">
      <cpix:Data>
        <pskc:Secret>
          <pskc:PlainValue>YmFzZTY0ZW5jb2RlZAo=</pskc:PlainValue>
        </pskc:Secret>
      </cpix:Data>
    </cpix:ContentKey>
  </cpix:ContentKeyList>
  <cpix:DRMSystemList>
    <cpix:DRMSystem kid="11111111-1111-1111-1111-111111111111" systemId="9a04f079-9840-4286-ab92-e65be0885f95">
      <cpix:PSSH>YmFzZTY0ZW5jb2RlZAo=</cpix:PSSH>
    </cpix:DRMSystem>
    <cpix:DRMSystem kid="11111111-1111-1111-1111-111111111111" systemId="edef8ba9-79d6-4ace-a3c8-27dcd51d21ed">
      <cpix:PSSH>YmFzZTY0ZW5jb2RlZAo=</cpix:PSSH>
    </cpix:DRMSystem>
    <cpix:DRMSystem kid="11111111-1111-1111-1111-111111111111" systemId="94ce86fb-07ff-4f43-adb8-93d2fa968ca2">
      <cpix:URIExtXKey>YmFzZTY0ZW5jb2RlZAo=</cpix:URIExtXKey>
      <cpix:HLSSignalingData playlist="master">YmFzZTY0ZW5jb2RlZAo=</cpix:HLSSignalingData>
      <cpix:HLSSignalingData playlist="media">YmFzZTY0ZW5jb2RlZAo=</cpix:HLSSignalingData>
    </cpix:DRMSystem>
    <cpix:DRMSystem kid="22222222-2222-2222-22222-22222222222" systemId="9a04f079-9840-4286-ab92-e65be0885f95">
      <cpix:PSSH>YmFzZTY0ZW5jb2RlZAo=</cpix:PSSH>
    </cpix:DRMSystem>
    <cpix:DRMSystem kid="22222222-2222-2222-22222-22222222222" systemId="edef8ba9-79d6-4ace-a3c8-27dcd51d21ed">
      <cpix:PSSH>YmFzZTY0ZW5jb2RlZAo=</cpix:PSSH>
    </cpix:DRMSystem>
    <cpix:DRMSystem kid="22222222-2222-2222-22222-22222222222" systemId="94ce86fb-07ff-4f43-adb8-93d2fa968ca2">
      <cpix:URIExtXKey>YmFzZTY0ZW5jb2RlZAo=</cpix:URIExtXKey>
      <cpix:HLSSignalingData playlist="master">YmFzZTY0ZW5jb2RlZAo=</cpix:HLSSignalingData>
      <cpix:HLSSignalingData playlist="media">YmFzZTY0ZW5jb2RlZAo=</cpix:HLSSignalingData>
    </cpix:DRMSystem>
  </cpix:DRMSystemList>
  <cpix:ContentKeyUsageRuleList>
    <cpix:ContentKeyUsageRule kid="11111111-1111-1111-1111-111111111111">
      <cpix:VideoFilter></cpix:VideoFilter>
    </cpix:ContentKeyUsageRule>
    <cpix:ContentKeyUsageRule kid="22222222-2222-2222-22222-22222222222">
      <cpix:AudioFilter></cpix:AudioFilter>
    </cpix:ContentKeyUsageRule>
  </cpix:ContentKeyUsageRuleList>
</cpix:CPIX>
```

#### Multikey with audio clear

Get a CPIX 2.1 document that uses only 1 content key for video; audio tracks are left in the clear.

##### Optional query params:

* `drm` : comma separated list of DRM systems to be included in the response.

##### Request

```
curl -XGET -H "API-KEY: <api-key>" \
"https://cpix.vudrm.tech/v1/cpix/<client-name>/<content-id>/multikey/audioclear"
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<cpix:CPIX xmlns:pskc="urn:ietf:params:xml:ns:keyprov:pskc" xmlns:xsi="urn:ietf:params:xml:ns:keyprov:pskc" xmlns:cpix="urn:dashif:org:cpix" xsi:schemaLocation="urn:dashif:org:cpix cpix.xsd">
  <cpix:ContentKeyList>
    <cpix:ContentKey kid="11111111-1111-1111-1111-111111111111" explicitIV="YmFzZTY0ZW5jb2RlZAo=">
      <cpix:Data>
        <pskc:Secret>
          <pskc:PlainValue>YmFzZTY0ZW5jb2RlZAo=</pskc:PlainValue>
        </pskc:Secret>
      </cpix:Data>
    </cpix:ContentKey>
    <cpix:ContentKey kid="22222222-2222-2222-22222-22222222222">
    </cpix:ContentKey>
  </cpix:ContentKeyList>
  <cpix:DRMSystemList>
    <cpix:DRMSystem kid="11111111-1111-1111-1111-111111111111" systemId="9a04f079-9840-4286-ab92-e65be0885f95">
      <cpix:PSSH>YmFzZTY0ZW5jb2RlZAo=</cpix:PSSH>
    </cpix:DRMSystem>
    <cpix:DRMSystem kid="11111111-1111-1111-1111-111111111111" systemId="edef8ba9-79d6-4ace-a3c8-27dcd51d21ed">
      <cpix:PSSH>YmFzZTY0ZW5jb2RlZAo=</cpix:PSSH>
    </cpix:DRMSystem>
    <cpix:DRMSystem kid="11111111-1111-1111-1111-111111111111" systemId="94ce86fb-07ff-4f43-adb8-93d2fa968ca2">
      <cpix:URIExtXKey>YmFzZTY0ZW5jb2RlZAo=</cpix:URIExtXKey>
      <cpix:HLSSignalingData playlist="master">YmFzZTY0ZW5jb2RlZAo=</cpix:HLSSignalingData>
      <cpix:HLSSignalingData playlist="media">YmFzZTY0ZW5jb2RlZAo=</cpix:HLSSignalingData>
    </cpix:DRMSystem>
    <cpix:DRMSystem kid="22222222-2222-2222-22222-22222222222" systemId="9a04f079-9840-4286-ab92-e65be0885f95">
      <cpix:PSSH>YmFzZTY0ZW5jb2RlZAo=</cpix:PSSH>
    </cpix:DRMSystem>
    <cpix:DRMSystem kid="22222222-2222-2222-22222-22222222222" systemId="edef8ba9-79d6-4ace-a3c8-27dcd51d21ed">
      <cpix:PSSH>YmFzZTY0ZW5jb2RlZAo=</cpix:PSSH>
    </cpix:DRMSystem>
    <cpix:DRMSystem kid="22222222-2222-2222-22222-22222222222" systemId="94ce86fb-07ff-4f43-adb8-93d2fa968ca2">
      <cpix:URIExtXKey>YmFzZTY0ZW5jb2RlZAo=</cpix:URIExtXKey>
      <cpix:HLSSignalingData playlist="master">YmFzZTY0ZW5jb2RlZAo=</cpix:HLSSignalingData>
      <cpix:HLSSignalingData playlist="media">YmFzZTY0ZW5jb2RlZAo=</cpix:HLSSignalingData>
    </cpix:DRMSystem>
  </cpix:DRMSystemList>
  <cpix:ContentKeyUsageRuleList>
    <cpix:ContentKeyUsageRule kid="11111111-1111-1111-1111-111111111111">
      <cpix:VideoFilter></cpix:VideoFilter>
    </cpix:ContentKeyUsageRule>
    <cpix:ContentKeyUsageRule kid="22222222-2222-2222-22222-22222222222">
      <cpix:AudioFilter></cpix:AudioFilter>
    </cpix:ContentKeyUsageRule>
  </cpix:ContentKeyUsageRuleList>
</cpix:CPIX>
```

#### Multikey with seperate bitrates based key

Get a CPIX 2.1 document that uses different content keys per bitrate and one for audio.
If provided with a single bitrate the document will use one key for all bitrates up to the given bitrate, and one key for the given bitrate up.
If provided with two bitrates the document will use one key for all bitrates up to the lowest given bitrate, one key for all bitrates between the lowest and highest bitrates, and one key for the highest given bitrate up.

##### Query params:

* `bitrates` : a single bitrate, or two seperated by a comma.

##### Optional query params:

* `drm` : comma separated list of DRM systems to be included in the response.

##### Request

```
curl -XGET -H "API-KEY: <api-key>" \
"https://cpix.vudrm.tech/v1/cpix/<client-name>/<content-id>/multikey?bitrates=8"
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<cpix:CPIX xmlns:pskc="urn:ietf:params:xml:ns:keyprov:pskc" xmlns:xsi="urn:ietf:params:xml:ns:keyprov:pskc" xmlns:cpix="urn:dashif:org:cpix" xmlns:speke="urn:aws:amazon:com:speke" xsi:schemaLocation="urn:dashif:org:cpix cpix.xsd">
    <cpix:ContentKeyList>
        <cpix:ContentKey kid="11111111-1111-1111-1111-111111111111" explicitIV="YmFzZTY0ZW5jb2RlZAo=">
            <cpix:Data>
                <pskc:Secret>
                    <pskc:PlainValue>YmFzZTY0ZW5jb2RlZAo=</pskc:PlainValue>
                </pskc:Secret>
            </cpix:Data>
        </cpix:ContentKey>
        <cpix:ContentKey kid="22222222-2222-2222-22222-22222222222" explicitIV="YmFzZTY0ZW5jb2RlZAo=">
            <cpix:Data>
                <pskc:Secret>
                    <pskc:PlainValue>YmFzZTY0ZW5jb2RlZAo=</pskc:PlainValue>
                </pskc:Secret>
            </cpix:Data>
        </cpix:ContentKey>
        <cpix:ContentKey kid="33333333-3333-3333-33333-33333333333" explicitIV="YmFzZTY0ZW5jb2RlZAo=">
            <cpix:Data>
                <pskc:Secret>
                    <pskc:PlainValue>YmFzZTY0ZW5jb2RlZAo=</pskc:PlainValue>
                </pskc:Secret>
            </cpix:Data>
        </cpix:ContentKey>
    </cpix:ContentKeyList>
    <cpix:DRMSystemList>
        <cpix:DRMSystem kid="11111111-1111-1111-1111-111111111111" systemId="9a04f079-9840-4286-ab92-e65be0885f95">
            <cpix:PSSH>YmFzZTY0ZW5jb2RlZAo=</cpix:PSSH>
        </cpix:DRMSystem>
        <cpix:DRMSystem kid="11111111-1111-1111-1111-111111111111" systemId="edef8ba9-79d6-4ace-a3c8-27dcd51d21ed">
            <cpix:PSSH>YmFzZTY0ZW5jb2RlZAo=</cpix:PSSH>
        </cpix:DRMSystem>
        <cpix:DRMSystem kid="11111111-1111-1111-1111-111111111111" systemId="94ce86fb-07ff-4f43-adb8-93d2fa968ca2">
            <cpix:URIExtXKey>YmFzZTY0ZW5jb2RlZAo=</cpix:URIExtXKey>
            <cpix:HLSSignalingData playlist="master">YmFzZTY0ZW5jb2RlZAo=</cpix:HLSSignalingData>
            <cpix:HLSSignalingData playlist="media">YmFzZTY0ZW5jb2RlZAo=</cpix:HLSSignalingData>
        </cpix:DRMSystem>
        <cpix:DRMSystem kid="22222222-2222-2222-22222-22222222222" systemId="9a04f079-9840-4286-ab92-e65be0885f95">
            <cpix:PSSH>YmFzZTY0ZW5jb2RlZAo=</cpix:PSSH>
        </cpix:DRMSystem>
        <cpix:DRMSystem kid="22222222-2222-2222-22222-22222222222" systemId="edef8ba9-79d6-4ace-a3c8-27dcd51d21ed">
            <cpix:PSSH>YmFzZTY0ZW5jb2RlZAo=</cpix:PSSH>
        </cpix:DRMSystem>
        <cpix:DRMSystem kid="22222222-2222-2222-22222-22222222222" systemId="94ce86fb-07ff-4f43-adb8-93d2fa968ca2">
            <cpix:URIExtXKey>YmFzZTY0ZW5jb2RlZAo=</cpix:URIExtXKey>
            <cpix:HLSSignalingData playlist="master">YmFzZTY0ZW5jb2RlZAo=</cpix:HLSSignalingData>
            <cpix:HLSSignalingData playlist="media">YmFzZTY0ZW5jb2RlZAo=</cpix:HLSSignalingData>
        </cpix:DRMSystem>
        <cpix:DRMSystem kid="33333333-3333-3333-33333-33333333333" systemId="9a04f079-9840-4286-ab92-e65be0885f95">
            <cpix:PSSH>YmFzZTY0ZW5jb2RlZAo=</cpix:PSSH>
        </cpix:DRMSystem>
        <cpix:DRMSystem kid="33333333-3333-3333-33333-33333333333" systemId="edef8ba9-79d6-4ace-a3c8-27dcd51d21ed">
            <cpix:PSSH>YmFzZTY0ZW5jb2RlZAo=</cpix:PSSH>
        </cpix:DRMSystem>
        <cpix:DRMSystem kid="33333333-3333-3333-33333-33333333333" systemId="94ce86fb-07ff-4f43-adb8-93d2fa968ca2">
            <cpix:URIExtXKey>YmFzZTY0ZW5jb2RlZAo=</cpix:URIExtXKey>
            <cpix:HLSSignalingData playlist="master">YmFzZTY0ZW5jb2RlZAo=</cpix:HLSSignalingData>
            <cpix:HLSSignalingData playlist="media">YmFzZTY0ZW5jb2RlZAo=</cpix:HLSSignalingData>
        </cpix:DRMSystem>
    </cpix:DRMSystemList>
    <cpix:ContentKeyPeriodList></cpix:ContentKeyPeriodList>
    <cpix:ContentKeyUsageRuleList>
        <cpix:ContentKeyUsageRule kid="11111111-1111-1111-1111-111111111111">
            <cpix:BitrateFilter maxBitrate="8"></cpix:BitrateFilter>
        </cpix:ContentKeyUsageRule>
        <cpix:ContentKeyUsageRule kid="22222222-2222-2222-22222-22222222222">
            <cpix:BitrateFilter minBitrate="8"></cpix:BitrateFilter>
        </cpix:ContentKeyUsageRule>
        <cpix:ContentKeyUsageRule kid="33333333-3333-3333-33333-33333333333">
            <cpix:AudioFilter></cpix:AudioFilter>
        </cpix:ContentKeyUsageRule>
    </cpix:ContentKeyUsageRuleList>
</cpix:CPIX>
```

#### Multikey with seperate video quality based keys

Get a CPIX 2.1 document that uses different content keys per video qualty and one for audio.
If provided with multiple video qualities the response will contain a encryption key for each video quality and one encrypt key for everything else.

##### Query params:

* `videoTracks` : a list of video qualities seperated by a comma, can only contain `SD`, `HD`, or `UHD`.

##### Optional query params:

* `drm` : comma separated list of DRM systems to be included in the response.

##### Request

```
curl -XGET -H "API-KEY: <api-key>" \
"https://cpix.vudrm.tech/v1/cpix/<client-name>/<content-id>/multikey?videoTracks=SD,HD"
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<cpix:CPIX xmlns:pskc="urn:ietf:params:xml:ns:keyprov:pskc" xmlns:xsi="urn:ietf:params:xml:ns:keyprov:pskc" xmlns:cpix="urn:dashif:org:cpix" xmlns:speke="urn:aws:amazon:com:speke" xsi:schemaLocation="urn:dashif:org:cpix cpix.xsd">
    <cpix:ContentKeyList>
        <cpix:ContentKey kid="11111111-1111-1111-1111-111111111111" explicitIV="YmFzZTY0ZW5jb2RlZAo=">
            <cpix:Data>
                <pskc:Secret>
                    <pskc:PlainValue>YmFzZTY0ZW5jb2RlZAo=</pskc:PlainValue>
                </pskc:Secret>
            </cpix:Data>
        </cpix:ContentKey>
        <cpix:ContentKey kid="22222222-2222-2222-22222-22222222222" explicitIV="YmFzZTY0ZW5jb2RlZAo=">
            <cpix:Data>
                <pskc:Secret>
                    <pskc:PlainValue>YmFzZTY0ZW5jb2RlZAo=</pskc:PlainValue>
                </pskc:Secret>
            </cpix:Data>
        </cpix:ContentKey>
        <cpix:ContentKey kid="33333333-3333-3333-33333-33333333333" explicitIV="YmFzZTY0ZW5jb2RlZAo=">
            <cpix:Data>
                <pskc:Secret>
                    <pskc:PlainValue>YmFzZTY0ZW5jb2RlZAo=</pskc:PlainValue>
                </pskc:Secret>
            </cpix:Data>
        </cpix:ContentKey>
        <cpix:ContentKey kid="44444444-4444-4444-44444-44444444444" explicitIV="YmFzZTY0ZW5jb2RlZAo=">
            <cpix:Data>
                <pskc:Secret>
                    <pskc:PlainValue>YmFzZTY0ZW5jb2RlZAo=</pskc:PlainValue>
                </pskc:Secret>
            </cpix:Data>
        </cpix:ContentKey>
    </cpix:ContentKeyList>
    <cpix:DRMSystemList>
        <cpix:DRMSystem kid="11111111-1111-1111-1111-111111111111" systemId="9a04f079-9840-4286-ab92-e65be0885f95">
            <cpix:PSSH>YmFzZTY0ZW5jb2RlZAo=</cpix:PSSH>
        </cpix:DRMSystem>
        <cpix:DRMSystem kid="11111111-1111-1111-1111-111111111111" systemId="edef8ba9-79d6-4ace-a3c8-27dcd51d21ed">
            <cpix:PSSH>YmFzZTY0ZW5jb2RlZAo=</cpix:PSSH>
        </cpix:DRMSystem>
        <cpix:DRMSystem kid="11111111-1111-1111-1111-111111111111" systemId="94ce86fb-07ff-4f43-adb8-93d2fa968ca2">
            <cpix:URIExtXKey>YmFzZTY0ZW5jb2RlZAo=</cpix:URIExtXKey>
            <cpix:HLSSignalingData playlist="master">YmFzZTY0ZW5jb2RlZAo=</cpix:HLSSignalingData>
            <cpix:HLSSignalingData playlist="media">YmFzZTY0ZW5jb2RlZAo=</cpix:HLSSignalingData>
        </cpix:DRMSystem>
        <cpix:DRMSystem kid="22222222-2222-2222-22222-22222222222" systemId="9a04f079-9840-4286-ab92-e65be0885f95">
            <cpix:PSSH>YmFzZTY0ZW5jb2RlZAo=</cpix:PSSH>
        </cpix:DRMSystem>
        <cpix:DRMSystem kid="22222222-2222-2222-22222-22222222222" systemId="edef8ba9-79d6-4ace-a3c8-27dcd51d21ed">
            <cpix:PSSH>YmFzZTY0ZW5jb2RlZAo=</cpix:PSSH>
        </cpix:DRMSystem>
        <cpix:DRMSystem kid="22222222-2222-2222-22222-22222222222" systemId="94ce86fb-07ff-4f43-adb8-93d2fa968ca2">
            <cpix:URIExtXKey>YmFzZTY0ZW5jb2RlZAo=</cpix:URIExtXKey>
            <cpix:HLSSignalingData playlist="master">YmFzZTY0ZW5jb2RlZAo=</cpix:HLSSignalingData>
            <cpix:HLSSignalingData playlist="media">YmFzZTY0ZW5jb2RlZAo=</cpix:HLSSignalingData>
        </cpix:DRMSystem>
        <cpix:DRMSystem kid="33333333-3333-3333-33333-33333333333" systemId="9a04f079-9840-4286-ab92-e65be0885f95">
            <cpix:PSSH>YmFzZTY0ZW5jb2RlZAo=</cpix:PSSH>
        </cpix:DRMSystem>
        <cpix:DRMSystem kid="33333333-3333-3333-33333-33333333333" systemId="edef8ba9-79d6-4ace-a3c8-27dcd51d21ed">
            <cpix:PSSH>YmFzZTY0ZW5jb2RlZAo=</cpix:PSSH>
        </cpix:DRMSystem>
        <cpix:DRMSystem kid="33333333-3333-3333-33333-33333333333" systemId="94ce86fb-07ff-4f43-adb8-93d2fa968ca2">
            <cpix:URIExtXKey>YmFzZTY0ZW5jb2RlZAo=</cpix:URIExtXKey>
            <cpix:HLSSignalingData playlist="master">YmFzZTY0ZW5jb2RlZAo=</cpix:HLSSignalingData>
            <cpix:HLSSignalingData playlist="media">YmFzZTY0ZW5jb2RlZAo=</cpix:HLSSignalingData>
        </cpix:DRMSystem>
        <cpix:DRMSystem kid="44444444-4444-4444-44444-44444444444" systemId="9a04f079-9840-4286-ab92-e65be0885f95">
            <cpix:PSSH>YmFzZTY0ZW5jb2RlZAo=</cpix:PSSH>
        </cpix:DRMSystem>
        <cpix:DRMSystem kid="44444444-4444-4444-44444-44444444444" systemId="edef8ba9-79d6-4ace-a3c8-27dcd51d21ed">
            <cpix:PSSH>YmFzZTY0ZW5jb2RlZAo=</cpix:PSSH>
        </cpix:DRMSystem>
        <cpix:DRMSystem kid="44444444-4444-4444-44444-44444444444" systemId="94ce86fb-07ff-4f43-adb8-93d2fa968ca2">
            <cpix:URIExtXKey>YmFzZTY0ZW5jb2RlZAo=</cpix:URIExtXKey>
            <cpix:HLSSignalingData playlist="master">YmFzZTY0ZW5jb2RlZAo=</cpix:HLSSignalingData>
            <cpix:HLSSignalingData playlist="media">YmFzZTY0ZW5jb2RlZAo=</cpix:HLSSignalingData>
        </cpix:DRMSystem>
    </cpix:DRMSystemList>
    <cpix:ContentKeyPeriodList></cpix:ContentKeyPeriodList>
    <cpix:ContentKeyUsageRuleList>
        <cpix:ContentKeyUsageRule kid="11111111-1111-1111-1111-111111111111">
            <cpix:BitrateFilter maxPixels="409920"></cpix:BitrateFilter>
        </cpix:ContentKeyUsageRule>
        <cpix:ContentKeyUsageRule kid="22222222-2222-2222-22222-22222222222">
            <cpix:BitrateFilter minPixels="409921" maxPixels="2073600"></cpix:BitrateFilter>
        </cpix:ContentKeyUsageRule>
        <cpix:ContentKeyUsageRule kid="33333333-3333-3333-33333-33333333333">
            <cpix:VideoFilter minPixels="2073601"></cpix:VideoFilter>
        </cpix:ContentKeyUsageRule>
        <cpix:ContentKeyUsageRule kid="44444444-4444-4444-44444-44444444444">
            <cpix:AudioFilter></cpix:AudioFilter>
        </cpix:ContentKeyUsageRule>
    </cpix:ContentKeyUsageRuleList>
</cpix:CPIX>
```

#### Key Rotation

Get a CPIX 2.1 document with Fairplay keys at every given interval between two given times; key rotation is only supported by HLS with Fairplay or AES.

###### USP support 

Key rotation works with USP versions `1.10.12` and `1.10.18`. If you wish to use version `1.9.5` the `explicitIV` needs to be removed from each content key before the CPIX document is used. This can be done with the following command if the CPIX document has been stored in a file name `keys.cpix`.

```bash
sed -i -E 's/ explicitIV=.*"//' keys.cpix
```

##### Query params:

* `start` : start time in `yyyy-MM-ddThh-mm-ssZ` format. E.g. 1970-01-01T00:00:00Z
* `end` : end time in `yyyy-MM-ddThh-mm-ssZ` format. E.g. 1970-01-01T01:00:00Z
* `interval` : interval each key should last in seconds.

##### Optional query params:

* `drm` : comma separated list of DRM systems to be included in the response. **Can only be `fairplay` or `aes`**

##### Request

```bash
curl -XGET -H "API-KEY: <api-key>" \
"https://cpix-beta.eu-central-1.vudrm.tech/v1/cpix/<client-name>/<content-id>/rotate?start=<start-time>&end=<end-time>&interval=<interval>"
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<cpix:CPIX xmlns:pskc="urn:ietf:params:xml:ns:keyprov:pskc" xmlns:xsi="urn:ietf:params:xml:ns:keyprov:pskc" xmlns:cpix="urn:dashif:org:cpix" xmlns:speke="urn:aws:amazon:com:speke" xsi:schemaLocation="urn:dashif:org:cpix cpix.xsd">
  <cpix:ContentKeyList>
    <cpix:ContentKey kid="11111111-1111-1111-1111-111111111111" explicitIV="YmFzZTY0ZW5jb2RlZAo=">
      <cpix:Data>
        <pskc:Secret>
          <pskc:PlainValue>YmFzZTY0ZW5jb2RlZAo=</pskc:PlainValue>
        </pskc:Secret>
      </cpix:Data>
    </cpix:ContentKey>
    <cpix:ContentKey kid="22222222-2222-2222-22222-22222222222" explicitIV="YjIxZmRlNzI5MDUyMDEzYw==">
      <cpix:Data>
        <pskc:Secret>
          <pskc:PlainValue>YmFzZTY0ZW5jb2RlZAo=</pskc:PlainValue>
        </pskc:Secret>
      </cpix:Data>
    </cpix:ContentKey>
    <cpix:ContentKey kid="33333333-3333-3333-3333-333333333333" explicitIV="YmFzZTY0ZW5jb2RlZAo=">
      <cpix:Data>
        <pskc:Secret>
          <pskc:PlainValue>YmFzZTY0ZW5jb2RlZAo=</pskc:PlainValue>
        </pskc:Secret>
      </cpix:Data>
    </cpix:ContentKey>
  </cpix:ContentKeyList>
  <cpix:DRMSystemList>
    <cpix:DRMSystem kid="11111111-1111-1111-1111-111111111111" systemId="94ce86fb-07ff-4f43-adb8-93d2fa968ca2">
      <cpix:URIExtXKey>YmFzZTY0ZW5jb2RlZAo=</cpix:URIExtXKey>
      <cpix:HLSSignalingData>YmFzZTY0ZW5jb2RlZAo=</cpix:HLSSignalingData>
    </cpix:DRMSystem>
    <cpix:DRMSystem kid="22222222-2222-2222-22222-22222222222" systemId="94ce86fb-07ff-4f43-adb8-93d2fa968ca2">
      <cpix:URIExtXKey>YmFzZTY0ZW5jb2RlZAo=</cpix:URIExtXKey>
      <cpix:HLSSignalingData>YmFzZTY0ZW5jb2RlZAo=</cpix:HLSSignalingData>
    </cpix:DRMSystem>
    <cpix:DRMSystem kid="33333333-3333-3333-3333-333333333333" systemId="94ce86fb-07ff-4f43-adb8-93d2fa968ca2">
      <cpix:URIExtXKey>YmFzZTY0ZW5jb2RlZAo=</cpix:URIExtXKey>
      <cpix:HLSSignalingData>YmFzZTY0ZW5jb2RlZAo=</cpix:HLSSignalingData>
    </cpix:DRMSystem>
  </cpix:DRMSystemList>
  <cpix:ContentKeyPeriodList>
    <cpix:ContentKeyPeriod id="period_0" start="1970-01-01T00:00:00Z" end="1970-01-01T00:30:00Z"></cpix:ContentKeyPeriod>
    <cpix:ContentKeyPeriod id="period_1" start="1970-01-01T00:30:00Z" end="1970-01-01T01:00:00Z"></cpix:ContentKeyPeriod>
    <cpix:ContentKeyPeriod id="period_2" start="1970-01-01T01:00:00Z" end="1970-01-01T01:30:00Z"></cpix:ContentKeyPeriod>
  </cpix:ContentKeyPeriodList>
  <cpix:ContentKeyUsageRuleList>
    <cpix:ContentKeyUsageRule kid="11111111-1111-1111-1111-111111111111">
      <cpix:KeyPeriodFilter periodId="period_0"></cpix:KeyPeriodFilter>
    </cpix:ContentKeyUsageRule>
    <cpix:ContentKeyUsageRule kid="22222222-2222-2222-22222-22222222222">
      <cpix:KeyPeriodFilter periodId="period_1"></cpix:KeyPeriodFilter>
    </cpix:ContentKeyUsageRule>
    <cpix:ContentKeyUsageRule kid="33333333-3333-3333-3333-333333333333">
      <cpix:KeyPeriodFilter periodId="period_2"></cpix:KeyPeriodFilter>
    </cpix:ContentKeyUsageRule>
  </cpix:ContentKeyUsageRuleList>
</cpix:CPIX>
```

#### JSON Keys

Get encryption keys and license URLs for a given `<content-id>` as JSON.

##### Optional query params:

* `drm` : comma separated list of DRM systems to be included in the response.

##### Request

```
curl -XGET -H "API-KEY: <api-key>" \
"https://cpix.vudrm.tech/v1/keys/<client-name>/<content-id>"
```

```json
{
  "key_id_hex": "76616c7565686578666f726d61746564",
  "content_key_hex": "76616c7565686578666f726d61746564",
  "iv_hex": "76616c7565686578666f726d61746564",
  "playready_pssh_data": "YmFzZTY0LXBsYXlyZWFkeS1oZWFkZXItb2JqZWN0Cg==",
  "playready_system_id": "9a04f079-9840-4286-ab92-e65be0885f95",
  "widevine_drm_specific_data": "YmFzZTY0LXdpZGV2aW5lLXBzc2gtZGF0YQo=",
  "widevine_system_id": "edef8ba9-79d6-4ace-a3c8-27dcd51d21ed",
  "fairplay_system_id": "94ce86fb-07ff-4f43-adb8-93d2fa968ca2",
  "fairplay_laurl": "skd://fairplay-license.vudrm.tech/license/<content-id>",
}
```

The values are:

- `key_id_hex`: Unique ID for the encryption. Base16 in Little Endian. Used by PlayReady and Widevine.
- `content_key_hex`: 128bit encryption key in base16 in Little Endian. Used by Fairplay, PlayReady, and Widevine.
- `iv_hex`: The encryption key. Used by Fairplay.
- `playready_pssh_data`: Base 64 encoded pssh box containing the PlayReady header.
- `playready_system_id`: The DASH protection system-specific identifier for PlayReady.
- `widevine_drm_specific_data`: Base 64 encoded pssh box containing the content id.
- `widevine_system_id`: The DASH protection system-specific identifier for Widevine.
- `fairplay_system_id`: The DASH protection system-specific identifier for Fairplay.
- `fairplay_laurl`: The Fairplay license url.

## SPEKE Key Provider API
These are the docs for the speke key provider, which has a speke endpoint for use with AWS. To learn how to use our SPEKE API with AWS' Media Services please read our [guide](/projects/vudrm/en/latest/DeveloperDocumentation/aws.html).

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

## Legacy

### Legacy JSON Key provider API

The Legacy JSON Key Provider API will provide all the information required in order to encrypt various types of content. In order to be as flexible as possible it will return encryption keys in Base16 and Base64 as some systems require different values. 

The following sections describe the requests and responses in order to retrieve the keys. The most common scenario is to request CENC and Fairplay encryption keys. 

#### Request

A `GET` should be made to the following URL:

```URL
https://keyprovider.vudrm.tech/<DRM_TYPE>/<CLIENT_NAME>/<CONTENT_ID> 
```

The breakdown of this URL is:
- `DRM_TYPE`: The type of encryption keys being requested. Possible values are `cenc`, `fairplay`, `widevine`, `playready`, and `aes`. See [Using the encryption keys](#using-the-encryption-keys) for more information.
- `CLIENT_NAME`: The account name. Plese contact support@vualto.com if you do not have an account name.
- `CONTENT_ID`: A unique content identifier. This value will always generate the same encryption keys. May only contain alphanumeric characters, underscores, and hyphens.

In order to provide the keys securely, an API key is required in the header of the request. Please contact support@vualto.com if you do not have an API key.

Below is an example curl request for `cenc` encryption keys using the client `vualto` and with the `CONTENT_ID` of `test`.

```bash
curl -X "GET" "https://keyprovider.vudrm.tech/cenc/vualto/test" -H "API_KEY: <API_KEY>" 
```

#### Response

In order to provide the keys securely, the DRM encryption keys are encrypted in the response.

The format of the response will be:

```JSON
{
    "key":"r8tXpf8wSSHKVnW1fz+MgZY3BTnTO+/IGFglt9oUwtKWj8eyguSLd0bzOhcn4DMF4JWOAbhbKopJE/cZWPqIfl3RCIwl46UP57/m7870+z3NWxh/JSvrfWBq4SGmkykfDLKjyLBqb5F6dDlUB+flcfZMcQh6FGk5GNGEniXliB/kyCrVDiKdwAw3hft8rT4/itFQDMRKkhJf1Fh67AZ1LyzOI3CCY/oO4w/XF/XMAJ1z92tAKULI+cPMFaXT3I77N3iaQoYN78mJkKgAr71Tlf91+ASB8jqOCHD6nNgm6nGxZlsTVK5wxdhT4zbMJq4xb7daSy7xMWdH/1eXuIh0ZhWicKJqbWHKIeifZWFras/0QzqN27rupHF3UYnYQ40V3ESE2PUIe8Cs8471QRHlruYlwtgBBmGme2ERZWFjrRFEeUt8SobQkCIZrzDEKaQ2bJMyOy1yMt8oklpFaW9pBg3yHZEK1WZn5u8ynV+ZWLI6soCNpMgkPYe2UtnFOhFRZfT8WG09FGJiouzyL3fCZAguxP8u9jkzQTv15UhwO/StdpTKCn1yxr3KoyxGAk3L|166c43d4023880a99691fd9679e6ad785305f205"
} 
```

The value of `key` has two components separated by a pipe (ASCII code 124):
- Encrypted blob containing the DRM encryption keys. 
- Hash used in a checksum.

##### Decrypting the response

This section contains examples of how to decrypt the response from the Legacy JSON Key Provider API.

You will need the `CLIENT_NAME`, `SHARED_SECRET` and the `KEY_PROVIDER_RESPONSE` value from the request to the Legacy JSON Key Provider API.

###### C#

```c#
using System;
using System.IO;
using System.Security.Cryptography;
using System.Text;

namespace Vudrm.EncryptionExamples
{
    class Program
    {
        private const string Client = "CLIENT_NAME";
        private const string SharedSecret = "SHARED_SECRET";
        private const string KeyProviderResponse = "KEY_PROVIDER_RESPONSE";
        
        static void Main(string[] args)
        {
            Console.WriteLine("Decrypting keyprovider API response...");
            var splitKeyProviderResponse = KeyProviderResponse.Split('|');
            var computedHash = ComputeHash(SharedSecret, Client, splitKeyProviderResponse[0]);
            if (computedHash == splitKeyProviderResponse[1])
            {
                Console.WriteLine("key providers response hash passed validation");
                var decryptedKeyProviderResponse = Decrypt(splitKeyProviderResponse[0], SharedSecret);
                Console.WriteLine("Decrypted key provider response: " + decryptedKeyProviderResponse);
            }
            else
            {
                Console.WriteLine("Key provider response hash did not validate");
            }
            Console.ReadLine();
        }

        private string Decrypt(string policy, string sharedSecret)
        {
            try
            {
                var encrypted = Convert.FromBase64String(policy);
                var key = Encoding.ASCII.GetBytes(sharedSecret.Substring(0, 16));
                var rj = new RijndaelManaged
                {
                    Mode = CipherMode.ECB,
                    BlockSize = 128,
                    Key = key
                };
                var decryptor = rj.CreateDecryptor(key, null);
                var ms = new MemoryStream(encrypted);
                var cs = new CryptoStream(ms, decryptor, CryptoStreamMode.Read);
                var plain = new byte[encrypted.Length];
                var decryptcount = cs.Read(plain, 0, plain.Length);
                ms.Close();
                cs.Close();
                return (Encoding.UTF8.GetString(plain, 0, decryptcount));
            }
            catch
            {
                return policy;
            }
        }

        private string ComputeHash(string sharedSecret, string client, string encryptedMessage)
        {
            var preComputed = sharedSecret + client + encryptedMessage;
            var algorithm = new SHA1Managed();
            var bytesOf = Encoding.UTF8.GetBytes(preComputed);
            var hashOf = algorithm.ComputeHash(bytesOf);
            var result = new StringBuilder();
            foreach (var b in hashOf)
            {
                result.Append(b.ToString("x2").ToLower());
            }
            return result.ToString();
        }
    }
}
```

###### Ruby

```ruby
require 'base64'
require 'crypt/rijndael'
require 'digest'
require 'openssl'

client = "CLIENT_NAME"
shared_secret = "SHARED_SECRET"

encrypted_keys = "KEY_PROVIDER_RESPONSE"

message, message_hash = encrypted_keys.split('|')

pre_computed = client + message
computed_hash = Digest::SHA1.new.hexdigest(shared_secret + pre_computed)

if computed_hash == message_hash
  decoded_message = Base64.strict_decode64(message)
  decipher = OpenSSL::Cipher::Cipher.new('AES-128-ECB').decrypt
  decipher.key = shared_secret[0..15]
  unencrypted_keys = decipher.update(decoded_message) + decipher.final
else
  raise "Key provider response hash did not validate"
end
```

###### GO

```go
package main

import (
	"crypto/aes"
	"crypto/sha1"
	"encoding/base64"
	"errors"
	"fmt"
	"strings"
)

var (
	Client = "CLIENT"
	SharedSecret = "SHARED_SECRET"
	KeyProviderResponse = "KEY_PROVIDER_RESPONSE"
	PKCS5 = &pkcs5{}
)

type pkcs5 struct{}

func main() {
	s := strings.Split(KeyProviderResponse, "|")
	encryptedMessage, hash := s[0], s[1]
	generatedHash := generateKeyProviderHash(SharedSecret, Client, encryptedMessage)
	if generatedHash == hash {
		decryptedKeys := decrypt(encryptedMessage, SharedSecret)
		fmt.Printf(decryptedKeys)
	} else {
		fmt.Printf("hash failed validation")
	}
}

func generateKeyProviderHash(sharedSecret string, client string, encryptedMessage string) string {
	v := fmt.Sprintf("%s%s%s", sharedSecret, client, encryptedMessage)
	hasher := sha1.New()
	hasher.Write([]byte(v))
	return fmt.Sprintf("%x", hasher.Sum(nil))
}

func decrypt(encryptedData string, sharedSecret string) string {
	source, _ := base64.StdEncoding.DecodeString(encryptedData)
	key := []byte(sharedSecret)[0:16]

	cipher, _ := aes.NewCipher([]byte(key))
	buffer := make([]byte, len(source))
	size := 16

	for bs, be := 0, size; bs < len(source); bs, be = bs+size, be+size {
		cipher.Decrypt(buffer[bs:be], source[bs:be])
	}

	plainBytes, _ := PKCS5.unPadding(buffer, 16)
	return string(plainBytes)
}

func (p *pkcs5) unPadding(src []byte, blockSize int) ([]byte, error) {
	srcLen := len(src)
	paddingLen := int(src[srcLen-1])
	if paddingLen >= srcLen || paddingLen > blockSize {
		return nil, errors.New("padding size error")
	}
	return src[:srcLen-paddingLen], nil
}
```

###### Python

```python
import base64
import binascii
import json
import hashlib

from Crypto.Cipher import AES

client = "CLIENT"
shared_secret = "SHARED_SECRET"
key_provider_response = "KEY_PROVIDER_RESPONSE"

print("Decrypting Key Provider Response...")
split_key_provider_response = key_provider_response.split("|")
pre_computed = (shared_secret + client + split_key_provider_response[0]).encode("utf-8")
computed_hash = hashlib.sha1(pre_computed).hexdigest()
if computed_hash == split_key_provider_response[1]:
    key = shared_secret[0:16]
    decoded_text = base64.b64decode(split_key_provider_response[0])
    aes = AES.new(key.encode("utf8"), AES.MODE_ECB)
    padded_text = aes.decrypt(decoded_text)
    unpadded_text = padded_text[:-padded_text[-1]]
    decrypted_keys = json.loads(unpadded_text)
    print("Decrypted keys: " + json.dumps(decrypted_keys))
else:
    print("Hash validation failed for key provider response!")
```

### Using the encryption keys

The next section explains the various encryption keys provided by the Legacy JSON Key Provider API. Each set has a different use case. For use with USP products [CENC](#cenc) and [Fairplay](#fairplay) encryption keys are recommended. See the Unified Streaming Integration section below for more details on how to use `mp4split` with the Legacy JSON Key Provider API.

This section uses the Legacy JSON Key Provider API but the JSON keys retrieved from the CPIX key provider can be used in the same way.

##### CENC

CENC is the Common Encryption Scheme and it standardises encryption keys between different DRM systems. This allows a single set of encryption keys to be use to encrypt a single file using different DRM systems. VUDRM supports Widevine and PlayReady when generating CENC encryption keys.

Make the following request to the Legacy JSON Key Provider API in order to retrieve `cenc` Keys.

```bash
curl -X GET https://keyprovider.vudrm.tech/cenc/<CLIENT>/<CONTENT_ID> -H 'API_KEY: <API_KEY>'
```

The keys returned in this response can be used for PlayReady and Widevine scenarios. They allow a single piece of content to be used across multiple devices and browsers.

An example decrypted response would be:

```JSON
{
    "key_id_big": "qMTumUz5drgbusWY+fi5LA==",
    "key_id_hex": "A8C4EE994CF976B81BBAC598F9F8B92C",
    "content_key": "ETweRxyDIuDqAadsWdx0HQ==",
    "content_key_hex": "113C1E471C8322E0EA01A76C59DC741D",
    "playready_key_iv": "dd80b66b2fe44be4",
    "playready_laurl": "https://playready-license.vudrm.tech/rightsmanager.asmx",
    "playready_checksum": "j+7cluk2J58=",
    "widevine_drm_specific_data": "Ig1zb21lY29udGVudGlkSOPclZsG",
    "widevine_laurl": "https://widevine-license.vudrm.tech/proxy"
}
```

The values are:

- `key_id_big`: Unique ID for the encryption. Base64 in Big Endian. 
- `key_id_hex`: Unique ID for the encryption. Base16 in Little Endian. This one should be used with `mp4split`.
- `content_key`: 128bit encryption key in base64 in Big Endian.
- `content_key_hex`: 128bit encryption key in base16 in Little Endian. This one should be used with `mp4split`.
- `playready_key_iv`: Additional random value to strengthen the PlayReady encryption.
- `playready_laurl`: The PlayReady license server URL.
- `playready_checksum`: A PlayReady specific security value required by some older PlayReady solutions and [Wowza](#wowza-integration).
- `widevine_drm_specific_data`: The Widevine PSSH box.
- `widevine_laurl`: The Widevine license server URL.

##### Fairplay

[Fairplay](https://developer.apple.com/streaming/fps/) is Apple's DRM system and is commonly used in conjunction with CENC encryption to provide support to the widest amount of devices possible.

Make the following request to retrieve `fairplay` keys from the Legacy JSON Key Provider API:

```bash
curl -X GET https://keyprovider.vudrm.tech/fairplay/<CLIENT>/<CONTENT_ID> -H 'API_KEY: <API_KEY>'
```

An example decrypted response would be:

```json
{
    "key_hex": "457A8E3300DE6D549A95037F1C7ADEB1",
    "iv_hex": "EB86EAFBD487391383E8FFF957561B0C",
    "laurl": "skd://fairplay-license.vudrm.tech/license/somecontentid"
}
```

The values are:
- `key_hex`: Unique ID for the encryption.
- `iv_hex`: The encryption key.
- `laurl`: The Fairplay license server URL. At the point of the request to the license server the `skd` protocol should be replaced with `https`. 

##### HLS AES encryption

HLS AES-128 is the Advanced Encryption Standard using a 128 bit key, Cipher Block Chaining (CBC) and PKCS7 padding.

Make the following request to retrieve HLS AES encryption keys from the Legacy JSON Key Provider API:

```bash
curl -X GET https://keyprovider.vudrm.tech/aes/<CLIENT>/<CONTENT_ID> -H 'API_KEY: <API_KEY>'
```

An example decrypted response would be:

```JSON
{
    "key_hex":"dd682004123622a99c2a2afcdad6217c",
    "key_url":"http://keyprovider.vudrm.tech:9293/aes/getkey/vualto/somecontentid"
}
```

The values are:
- `key_hex`: Base16 AES Content key
- `key_url`: URL to the AES Key Server 


##### PlayReady

[PlayReady](https://www.microsoft.com/playready/) is Microsoft's DRM system. Only use these keys directly to target PlayReady enabled devices. The use of CENC keys is preferred.

Make the following request to retrieve `playready` keys from the Legacy JSON Key Provider API:

```bash
curl -X GET https://keyprovider.vudrm.tech/playready/<CLIENT>/<CONTENT_ID> -H 'API_KEY: <API_KEY>'
```

An example decrypted response would be:

```JSON    
{
    "key_id":"kX9efHkfIS++X+m8kZPDOw==",
    "key_id_guid":"7c5e7f91-1f79-2f21-be5f-e9bc9193c33b",
    "key_id_uuid":"917f5e7c791f-2f21-be5fe9bc9193c33b",
    "key_id_hex":"917F5E7C791F212FBE5FE9BC9193C33B",
    "content_key":"eI5JjujIo1Ek7lqO+3gg0A==",
    "content_key_hex":"788E498EE8C8A35124EE5A8EFB7820D0",
    "laurl":"http://vualto.playready-license.vudrm.tech/rightsmanager.asmx",
    "service_id":"gwICI8yfIUGf4R/5qOWuqg==",
    "service_id_guid":"23020283-9fcc-4121-9fe11ff9a8e5aeaa",
    "key_iv":"07261ee6ee074f1d",
    "checksum":"wf0goCGB2O4="
}
```

The values are:
- `key_id`: Unique ID for the encryption. Base64 in Big Endian.
- `key_id_guid`: Unique ID for the encryption. GUID in Big Endian.
- `key_id_uuid`: Unique ID for the encryption. UUID in Little Endian.
- `key_id_hex`: Unique ID for the encryption. Base16 in Little Endean. This one should be used with `mp4split`.
- `content_key`: 128bit encryption key in base64.
- `content_key_hex`: 128bit encryption key in base16. This one should be used with `mp4split`.
- `laurl`: The PlayReady license server URL.
- `service_id`: Unique VUDRM service ID for Vualto in Base64.
- `service_id_guid`: Unique VUDRM service ID for Vualto as a GUID.
- `key_iv`: Additional random value to strengthen the PlayReady encryption.
- `checksum`: Extra security value required by some older PlayReady solutions.

##### Widevine 

[Widevine](https://www.widevine.com/) is Google's DRM system. Use these keys to target Widevine enabled devices directly. The use of CENC keys is preferred.

Make the following request to retrieve `widevine` keys from the Legacy JSON Key Provider API:

```bash
curl -X GET https://keyprovider.vudrm.tech/widevine/<CLIENT>/<CONTENT_ID> -H 'API_KEY: <API_KEY>'
```

An example decrypted response would be:

```JSON
{
    "key_id":"NxB8uJFHVSuaO5BjiMzuEQ==",
    "key_id_hex":"37107CB89147552B9A3B906388CCEE11",
    "content_key":"Unyx1s3fMFUedA688fCNxw==",
    "content_key_hex":"527CB1D6CDDF30551E740EBCF1F08DC7",
    "laurl":"https://widevine-license.vudrm.tech/proxy",
    "drm_specific_data":"CAESEDcQfLiRR1UrmjuQY4jM7hEaBnZ1YWx0byIFdGVzdDEqAkhEMgA="
} 
```

The values are:
- `key_id`: Unique ID for the encryption. Base64 in Little Endian.
- `key_id_hex`: Unique ID for the encryption. Base16 in Little Endian. This one should be used with `mp4split`.
- `content_key`: 128bit encryption key in base64.
- `content_key_hex`: 128bit encryption key in base16. This one should be used with `mp4split`.
- `laurl`: The Widevine license server URL.
- `drm_specific_data`: The Widevine PSSH box.

### Unified Streaming Integration

The encryption keys provided by the Legacy JSON Key Provider APIs are compatible with [Unified Streaming Platform's](https://www.unified-streaming.com/) mp4split product.

The recommended approach is to call the CPIX Key Provider API to retrieve CENC and Fairplay Keys. Once the encryption keys have been retrieved they can be used with mp4split to generate an ism:

```bash
mp4split --license-key=$LICENSE_KEY -o $ISM \
 --iss.key=${key_id_hex}:${content_key_hex} \
 --iss.key_iv=${playready_key_iv} \
 --iss.license_server_url=${playready_laurl} \
 --widevine.key=${key_id_hex}:${content_key_hex} \
 --widevine.license_server_url=${widevine_laurl} \
 --widevine.drm_specific_data=${widevine_drm_specific_data} \
 --hls.client_manifest_version=4 \
 --hls.key=:${key_hex} \
 --hls.key_iv=${iv_hex} \
 --hls.license_server_url=${laurl} \
 --hls.playout=sample_aes_streamingkeydelivery
```

The HLS values come from the Fairplay encryption keys, all other values are from the CENC keys.