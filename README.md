# resumableS3 (rs3)
## To solve what problem
When using S3 cp to download a very large file, original [AWS S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html) may fail and just report an error `("Connection broken: ConnectionResetError(104, 'Connection reset by peer')", ConnectionResetError(104, 'Connection reset by peer'))`, without writing the file to output, **which means the download of original AWS S3 is not resumable**.

ResumableS3 (rs3) is a Python based download utility, of which is meant to be a resumable download program. It is also multi-threaded, which means you shall get similar download speed as the original AWS S3.


## Installation
Use pip to install, <mark> you should have Python >= 3.8 installation first </mark>
```bash
conda create -n rs3 python=3.8 -y;
conda activate rs3;
pip install resumables3;
```



## Usage
Only two parameter is mandatory: `-i/--input` and `-o/--output`
```
Usage: rs3 [OPTIONS]

Options:
  -i, --input TEXT       S3 link, must be a specific downloadable object [required]
  -o, --output TEXT      Path to output  [required]
  -t, --temp TEXT        Path to the record file, default : download_progress.txt in output directory
  -w, --workers INTEGER  Max workers for download, default: max CPU threads in your system
  --chunk-size INTEGER   Chunk size for parallel download in MB [default: 25]
  --id TEXT              AWS access key id, default: None (anonymous)
  --key TEXT             AWS secert access key, default: None (anonymous)
  --region-name TEXT     AWS region name, default: None
  --version              Show the version and exit.
  --help                 Show this message and exit.
```

## Example
```
rs3 \
-i s3://human-pangenomics/working/HPRC_PLUS/HG01109/assemblies/year1_freeze_assembly_v2/HG01109.maternal.f1_assembly_v2.fa.gz \
-o ./output 
```


## How to resume download
When you use `rs3` to download file, you may encounter bad Internet connection for several chunk during `rs3` download (such as the example below), but it is totally fine.
```
Downloading sectors:  85%|█████████████████████████████████████████████████████████████████████████████████▏              | 36779/43519 [27:27:54<7:41:26,  4.11s/sector]
An error occurred while downloading chunk 964585062400-964611276799: An error occurred while reading from response stream: ('Connection broken: IncompleteRead(18505070 bytes read, 7709330 more expected)', IncompleteRead(18505070 bytes read, 7709330 more expected))
Downloading sectors:  87%|███████████████████████████████████████████████████████████████████████████████████▋            | 37960/43519 [28:16:53<6:08:05,  3.97s/sector]
An error occurred while downloading chunk 996225843200-996252057599: An error occurred while reading from response stream: ("Connection broken: ConnectionResetError(104, 'Connection reset by peer')", ConnectionResetError(104, 'Connection reset by peer'))
Downloading sectors: 100%|█████████████████████████████████████████████████████████████████████████████████████████████████▉| 43514/43519 [32:06:58<00:15,  3.10s/sector]
File downloaded to ./HG02723/HG02723_3.fast5.tar.gz
```

After the first `rs3` command finished, just rerun **the same rs3 command** and `rs3` will try to redownload the missing part: 
```
Downloading sectors: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████| 201/201 [08:00<00:00,  2.39s/sector]
File downloaded to ./HG02723/HG02723_3.fast5.tar.gz
```


## Validation
`rs3` and `s3` are used to download the same file: `s3://human-pangenomics/working/HPRC_PLUS/HG01109/assemblies/year1_freeze_assembly_v2/HG01109.maternal.f1_assembly_v2.fa.gz` with the following command:
```bash
# resumable S3
rs3 -i s3://human-pangenomics/working/HPRC_PLUS/HG01109/assemblies/year1_freeze_assembly_v2/HG01109.maternal.f1_assembly_v2.fa.gz -o ./rs3;

# AWS S3
aws s3 --no-sign-request cp s3://human-pangenomics/working/HPRC_PLUS/HG01109/assemblies/year1_freeze_assembly_v2/HG01109.maternal.f1_assembly_v2.fa.gz ./s3;
```
The downloaded files are compared through `md5sum`, which is exactly the same.
```
f2d8b690d0adeaf28ed3221514f30357  rs3/HG01109.maternal.f1_assembly_v2.fa.gz
f2d8b690d0adeaf28ed3221514f30357  s3/HG01109.maternal.f1_assembly_v2.fa.gz
```


## Citation
Just cite this repository.

