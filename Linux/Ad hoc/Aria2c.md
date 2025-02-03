aria2c -s 8 -x 8 -k 1024M  -d /mnt/data/ -o LiteNode_output-directory.tgz --retry-wait 1 --max-tries 0  --http-proxy 107.173.149.78:1088 http://34.86.86.229/backup20240903/LiteFullNode_output-directory.tgz

### pipeline download and extract

[](https://github.com/48Club/bsc-snapshots#pipeline-download-and-extract)

```shell
wget $Link -O - | zstd -cd | tar xf -

```



In the command:

bash

CopyEdit

`wget -qO- <URL> | tar -xz -C /path/to/destination`

the `-qO-` flags in `wget` mean:

- **`-q` (quiet mode)**: Suppresses the output, so `wget` doesn't print progress messages.
- **`-O-` (output to stdout instead of file)**: Instead of saving the downloaded file to disk, it outputs the data to `stdout` (standard output), which is then piped (`|`) to `tar`.