aria2c -s 8 -x 8 -k 1024M  -d /mnt/data/ -o LiteNode_output-directory.tgz --retry-wait 1 --max-tries 0  --http-proxy 107.173.149.78:1088 http://34.86.86.229/backup20240903/LiteFullNode_output-directory.tgz

### pipeline download and extract

[](https://github.com/48Club/bsc-snapshots#pipeline-download-and-extract)

```shell
wget $Link -O - | zstd -cd | tar xf -
```