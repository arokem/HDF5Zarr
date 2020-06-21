<strong>Reading HDF5 files with Zarr</strong>

## Installation

Requires latest dev installation of h5py


```bash
$ pip install git+https://github.com/catalystneuro/allen-institute-neuropixel-utils
```


## Usage:

```python

import zarr
from hdf5zarr import HDF5Zarr, NWBZARRHDF5IO
from hdf5zarr import rewrite_vlen_to_fixed

file_name = 'ecephys.nwb'

# Optional, if hdf5 file contains variable-length string datasets, rewrite them as fixed-length
rewrite_vlen_to_fixed(file_name)

# Local read
store = zarr.DirectoryStore('storezarr')
hdf5_zarr = HDF5Zarr(filename = file_name, store=store, store_mode='w', max_chunksize=2*2**20)
# Without indicating a specific zarr store, zarr uses the default zarr.MemoryStore
# alternatively pass a zarr store such as:
# store = zarr.DirectoryStore('storezarr')
# hdf5_zarr = HDF5Zarr(file_name, store = store, store_mode = 'w')
zgroup = hdf5_zarr.consolidate_metadata(metadata_key = '.zmetadata')
io = NWBZARRHDF5IO(mode='r+', file=zgroup, load_namespaces=True)             


# print dataset names
zgroup.tree()
# read
arr = zgroup['units/spike_times']
val = arr[0:1000]

# export metadata from zarr store to a single json file
import json
metadata_file = 'metadata'
with open(metadata_file, 'w') as f:
    json.dump(zgroup.store.meta_store, f)

# Remote read
import s3fs
# Set up S3 access
fs = s3fs.S3FileSystem()

# import metadata from a json file
with open(metadata_file, 'r') as f:
    metadata_dict = json.load(f)

store = metadata_dict
with fs.open('bucketname/' + file_name, 'rb') as f:
    hdf5_zarr = HDF5Zarr(f, store = store, store_mode = 'r')
    zgroup = hdf5_zarr.zgroup
    # print dataset names
    zgroup.tree()
    arr = zgroup['units/spike_times']
    val = arr[0:1000]

```