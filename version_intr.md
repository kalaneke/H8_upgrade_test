## version.josn 文件格式说明

#### *****代码格式*****

```json
{
    "latest_version": "v2.2.0",
    "release_date": "2026-04-15",
    "releases": [
        {
            "version": "v2.2.0",
            "type": "test",
            "release_note": "新增功能X，修复问题Y",
            "iap_files": [
                {
                    "file": "firmware.iap",
                    "file": "H8firmware.iap"
                    "checksum": "sha256:xyz789...",
                    "size": 1048576
                }
            ],
            "res_manifest": {
                "images/icon_1.png": "md5:abc...",
                "audio/startup.mp3": "md5:def..."
            }
        }
    ]
}
```

#### *1111*
