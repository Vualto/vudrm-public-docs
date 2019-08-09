# CPIX API

You may use our CPIX API in order to fetch VUDRM encyrption keys on CPIX XML document format. Compatible with Unified Streaming (version 1.9.3 and above).

> Replace `<api-key>`, `<client-name>` and `<content-id>` with appropiate values.

> Available DRM systems [`fairplay`, `playready`, `widevine`].

## Endpoints

### Default

Get a CPIX 2.1 document with all DRM systems `<client-name>` is entitled to use.

#### optional query params:

* `drm` : comma separated list of DRM systems to be included included on the response.

#### Request

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

### Decrypt

Get a CPIX 2.1 document with only a single content key.

#### optional query params:

* `drm` : comma separated list of DRM systems to be included included on the response.

#### Request

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

### Multikey

Get a CPIX 2.1 document that uses separate keys for video and audio tracks. Only supported for DASH with PlayReady and Widevine.

#### optional query params:

* `drm` : comma separated list of DRM systems to be included included on the response.

#### Request

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
    <cpix:DRMSystem kid="22222222-2222-2222-22222-22222222222" systemId="9a04f079-9840-4286-ab92-e65be0885f95">
      <cpix:PSSH>YmFzZTY0ZW5jb2RlZAo=</cpix:PSSH>
    </cpix:DRMSystem>
    <cpix:DRMSystem kid="22222222-2222-2222-22222-22222222222" systemId="edef8ba9-79d6-4ace-a3c8-27dcd51d21ed">
      <cpix:PSSH>YmFzZTY0ZW5jb2RlZAo=</cpix:PSSH>
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

### Multikey with audio clear

Get a CPIX 2.1 document that use only 1 content key for video, audio tracks are left on the clear. supported for DASH with PlayReady and Widevine.

#### optional query params:

* `drm` : comma separated list of DRM systems to be included included on the response.

#### Request

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
    <cpix:DRMSystem kid="22222222-2222-2222-22222-22222222222" systemId="9a04f079-9840-4286-ab92-e65be0885f95">
      <cpix:PSSH>YmFzZTY0ZW5jb2RlZAo=</cpix:PSSH>
    </cpix:DRMSystem>
    <cpix:DRMSystem kid="22222222-2222-2222-22222-22222222222" systemId="edef8ba9-79d6-4ace-a3c8-27dcd51d21ed">
      <cpix:PSSH>YmFzZTY0ZW5jb2RlZAo=</cpix:PSSH>
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

### Keys

Get a encryption keys and license URLs for a given `<content-id>` (JSON response).

#### optional query params:

* `drm` : comma separated list of DRM systems to be included included on the response.

#### Request

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
  "fairplay_laurl": "skd://fairplay-license.vudrm.tech/license/<content-id>"
}
```

## Unified Streaming Integration

Use our [CPIX Edge Reverse](https://cloud.docker.com/u/vualto/repository/docker/vualto/cpix-proxy) Proxy in order to work with USP.

