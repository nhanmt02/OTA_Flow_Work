# Step 1: 
## subcribe data ở topic:
```$aws/things/mqttfx/jobs/$next/get/accepted```

# Step 2: 
## public data lên topic đó với cái data format như bên dưới
```$aws/things/mqttfx/jobs/$next/get```
```
{"clientToken":"123"}
```
nó sẽ gửi data có dạng như sao về topic ở bước 1
```
{
  "clientToken" : "123",
  "timestamp" : 1640148187,
  "execution" : {
    "jobId" : "AFR_OTA-GD32-007",
    "status" : "IN_PROGRESS",
    "queuedAt" : 1639124162,
    "startedAt" : 1639124496,
    "lastUpdatedAt" : 1639124496,
    "versionNumber" : 2,
    "executionNumber" : 1,
    "jobDocument" : {
      "afr_ota" : {
        "protocols" : [ "MQTT" ],
        "streamname" : "AFR_OTA-ed09f158-b222-4669-b811-451e36868c9a",
        "files" : [ {
          "filepath" : "/",
          "filesize" : 60,
          "fileid" : 0,
          "certfile" : "/",
          "sig-sha256-ecdsa" : "MEQCIGmtv4Kq2jOR5VakuxJulDcKtmjWtuLBxVT8xLOft4adAiBsOaWkLBeW7HFhAh8txD73TA+LL0Y+D4fILcuJiiuINg=="
        } ]
      }
    }
  }
}
```
# Step 3: 
## Subcribe topic:

```$aws/things/mqttfx/streams/AFR_OTA-ed09f158-b222-4669-b811-451e36868c9a/description/json```

*** AFR_OTA-ed09f158-b222-4669-b811-451e36868c9a : là streamname ở bản tin ở trên vừa mới nhận được
# Step 4:
## public to topic:
```$aws/things/mqttfx/streams/AFR_OTA-ed09f158-b222-4669-b811-451e36868c9a/describe/json```

với nội dung:
```
{
    "c": "ec944cfb-1e3c-49ac-97de-9dc4aaad0039"
}
```
trong đó:

"ec944cfb-1e3c-49ac-97de-9dc4aaad0039":  là token

sau đó mình sẽ nhận được 1 message ở topic step 3 có nội dung:
```
{
  "c" : "ec944cfb-1e3c-49ac-97de-9dc4aaad0039",
  "s" : 1,
  "d" : "Stream for OTAUpdate GD32-007",
  "r" : [ {
    "f" : 0,
    "z" : 60
  } ]
}
```

### c: là token hồi nãy mình gửi lên
### s: là version của nó
### d: là description của gói job
### f: là stream ID
### z: là kích thước của data job (new firmware)


# Step 5: 
## subcribe topic
```$aws/things/mqttfx/streams/AFR_OTA-ed09f158-b222-4669-b811-451e36868c9a/data/json``` 
==> (nếu nó được accept)

```$aws/things/mqttfx/streams/AFR_OTA-ed09f158-b222-4669-b811-451e36868c9a/rejected/json```
==> nếu nó bị từ chối

# Step 6: 
## public data lên topic
``$aws/things/mqttfx/streams/AFR_OTA-ed09f158-b222-4669-b811-451e36868c9a/get/json``
với nội dung:
```
{
    "c": "ec944cfb-1e3c-49ac-97de-9dc4aaad0039",
    "s": 1,
    "f": 0,
    "l": 256,
    "o": 0,
    "n": 1
}
```
### c: là token
### s: version
### f: stream ID
### l: kích thước mỗi block(range 256 - 131072)
### o: offset of the block 
### n: số block 


sau đó nó sẽ gửi về cho mình trong 
```$aws/things/mqttfx/streams/AFR_OTA-ed09f158-b222-4669-b811-451e36868c9a/data/json```
```
{
  "c" : "ec944cfb-1e3c-49ac-97de-9dc4aaad0039",
  "f" : 0,
  "i" : 0,
  "l" : 60,
  "p" : "ew0KICAgICJvcGVyYXRpb24iOiAiZWNobyIsDQogICAgImFyZ3MiOiBbIkhlbGxvIHdvcmxkISJdDQp9"
}
```
trong đó 
### "p":"ew0KICAgICJvcGVyYXRpb24iOiAiZWNobyIsDQogICAgImFyZ3MiOiBbIkhlbGxvIHdvcmxkISJdDQp9" chính là data đã được mã hóa dạng BASE64