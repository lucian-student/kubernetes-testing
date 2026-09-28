```
docker exec --privileged -i cillium-worker2 ctr --namespace=k8s.io images import --all-platforms --digests --snapshotter=overlayfs -
```
* docker exec
    * -i -> interactive
    * --privileged

```


ctr --namespace=k8s.io images import --all-platforms --digests --snapshotter=overlayfs -
```
* ctr - je command line tool, který komunikuje s containerd

# kind

https://github.com/kubernetes-sigs/kind

Okej nejdřív zjistím, jestli to je bug, nebo to můžu vyřešit i jinak.

# bug

```
docker inspect quay.io/cilium/cilium:v1.20.2
[
    {
        "Id": "sha256:2939231d0d3e3ebddcd80fffa168b7ddcc78fdf0dc864d1c8c126ff523c54f01",
        "RepoTags": [
            "quay.io/cilium/cilium:v1.20.2"
        ],
        "RepoDigests": [
            "quay.io/cilium/cilium@sha256:2939231d0d3e3ebddcd80fffa168b7ddcc78fdf0dc864d1c8c126ff523c54f01"
        ],
        "Comment": "buildkit.dockerfile.v0",
        "Created": "2026-09-15T20:28:05.052232245Z",
        "Config": {
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "HUBBLE_SERVER=unix:///var/run/cilium/hubble.sock",
                "INITSYSTEM=SYSTEMD"
            ],
            "Cmd": [
                "/usr/bin/cilium-dbg"
            ],
            "WorkingDir": "/home/cilium",
            "Labels": {
                "maintainer": "maintainer@cilium.io"
            },
            "ArgsEscaped": true
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 1003165673,
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:00b1db870163e936c5b8f6fe75f706af800f64ae9dfa5b340f2e3c7cfa28a6b2",
                "sha256:d7533ee53d1bc0727ae75a0616bbfa3a8db61a411632c0c5b2031bc21dfca14c",
                "sha256:20537255215693342b663fadb2f45766102bd549659ae8f4ce4b38e5601d89bb",
                "sha256:4864939c35448fba6ba7565b45b24520863aadccfb65b69d45e7c933f227cbb2",
                "sha256:24dc0c9ea6b745ee3030d024a4fe647842fef0d7b92340a72a9b48cff96e5ed5"
            ]
        },
        "Metadata": {
            "LastTagTime": "2026-09-28T20:10:35.136765906Z"
        },
        "Descriptor": {
            "mediaType": "application/vnd.oci.image.index.v1+json",
            "digest": "sha256:2939231d0d3e3ebddcd80fffa168b7ddcc78fdf0dc864d1c8c126ff523c54f01",
            "size": 647
        },
        "Identity": {
            "Pull": [
                {
                    "Repository": "quay.io/cilium/cilium"
                }
            ]
        }
    }
]

kind load docker-image quay.io/cilium/cilium:v1.20.2 --name cillium
Image: "quay.io/cilium/cilium:v1.20.2" with ID "sha256:2939231d0d3e3ebddcd80fffa168b7ddcc78fdf0dc864d1c8c126ff523c54f01" not yet present on node "cillium-control-plane", loading...
Image: "quay.io/cilium/cilium:v1.20.2" with ID "sha256:2939231d0d3e3ebddcd80fffa168b7ddcc78fdf0dc864d1c8c126ff523c54f01" not yet present on node "cillium-worker3", loading...
Image: "quay.io/cilium/cilium:v1.20.2" with ID "sha256:2939231d0d3e3ebddcd80fffa168b7ddcc78fdf0dc864d1c8c126ff523c54f01" not yet present on node "cillium-worker", loading...
Image: "quay.io/cilium/cilium:v1.20.2" with ID "sha256:2939231d0d3e3ebddcd80fffa168b7ddcc78fdf0dc864d1c8c126ff523c54f01" not yet present on node "cillium-worker2", loading...
ERROR: failed to load image: command "docker exec --privileged -i cillium-worker2 ctr --namespace=k8s.io images import --all-platforms --digests --snapshotter=overlayfs -" failed with error: exit status 1
Command Output: ctr: content digest sha256:02dd062a1f48a6ef52f50e7396117f9e7c82fbe9e6408dcaebe212257414bbeb: not found
```

 It's a known issue (see kind#3795 referenced in the docs)
❌ Not a KIND bug — it's a Docker containerd store compatibility issue with multi-platform images
✅ Easy workaround: Use the --platform flag when saving the image, then use kind load image-archive


```
 docker image save \
  --platform linux/amd64 \
  --output /tmp/cilium.tar \
  quay.io/cilium/cilium:v1.20.2
kind load image-archive /tmp/cilium.tar --name cillium
```

```
docker exec --privileged -it cillium-control-plane ctr --namespace=k8s.io images ls | grep -i cilium
quay.io/cilium/cilium-envoy@sha256:af7382699576b9e65e9184efa52eeca0b58aea70ad6e511bf260c91d9f740463 application/vnd.oci.image.index.v1+json                   sha256:af7382699576b9e65e9184efa52eeca0b58aea70ad6e511bf260c91d9f740463 73.1 MiB  linux/amd64,linux/arm64                                                      io.cri-containerd.image=managed                                 
quay.io/cilium/cilium:v1.20.2                                                                       application/vnd.oci.image.manifest.v1+json                sha256:9d308e3f7f05972b0b0604c40d2b0f08fa2f6a55084fef1e1aaadb469e430639 245.9 MiB linux/amd64                                                                  io.cri-containerd.image=managed                                 
quay.io/cilium/cilium@sha256:2939231d0d3e3ebddcd80fffa168b7ddcc78fdf0dc864d1c8c126ff523c54f01       application/vnd.oci.image.index.v1+json                   sha256:2939231d0d3e3ebddcd80fffa168b7ddcc78fdf0dc864d1c8c126ff523c54f01 245.9 MiB linux/amd64,linux/arm64                                                      io.cri-containerd.image=managed   
```