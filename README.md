[![CI](https://github.com/baggepinnen/LazyWAVFiles.jl/workflows/CI/badge.svg)](https://github.com/baggepinnen/LazyWAVFiles.jl/actions)
[![codecov](https://codecov.io/gh/baggepinnen/LazyWAVFiles.jl/branch/master/graph/badge.svg)](https://codecov.io/gh/baggepinnen/LazyWAVFiles.jl)

# LazyWAVFiles
This package let's you treat a wav-file on disk as an `AbstractArray`. Access to the data is lazy, i.e., nothing (but size) is read from the file until the array is indexed into. You can also specify a folder containing many wav-files and treat them all as a single large array! This lets you work using files that are too large to fit in memory. Some examples
```julia
using LazyWAVFiles, WAV

# Create some files to work with
d   = mktempdir()
a,b = randn(Float32,10), randn(Float32,10)
WAV.wavwrite(a, joinpath(d,"f1.wav"), Fs=8000)
WAV.wavwrite(b, joinpath(d,"f2.wav"), Fs=8000)

# Indexing into the array loads data from disk
f1 = LazyWAVFile(joinpath(d,"f1.wav")) # This command only reads the size of the file.
f1[1]   == a[1]
f1[1:5] == a[1:5]
f1.fs   == 8000
size(f1)

# We can create an array from all files in a folder
df = DistributedWAVFile(d)       # This reads the size from all files.
df[1]    == a[1]                 # Indexing works the same
df[1:12] == [a; b[1:2]]          # We can even index over both arrays
df[:]    == [a;b]                # Or load all files as one long vector
df.fs    == 8000

size(df) # Other array functions are defined as well
length(df)

# To work using chunks of the entire distributed array, we can use Iterators.partition
julia> Iterators.partition(df, 2) |> collect
10-element Array{Array{Float32,1},1}:
 [0.44920132, -1.1176418]
 [-2.0420709, 0.11797007]
 [1.4723421, -0.32837275]
 [2.3656073, 0.4933495]   
 [-1.0910473, -0.18483315]
 [-0.5574947, -0.46916208]
 [0.27721304, -0.39077175]
 [-0.05172622, -0.715703]
 [0.5821298, 1.6757511]   
 [1.0726295, 0.23483518]
```


### Notes
- Creating a distributed file based on a folder with a really large number of files can take a while due to the size of each audio clip being read from each file. The size information is required in order to have the files appear as one large array. As an example:
```julia
julia> @time df = DistributedWAVFile("folder_with_21551_files/")
 25.518655 seconds (2.47 M allocations: 144.085 MiB, 0.18% gc time)
 DistributedWAVFile{Float32, 1} with 21551 files, 657735677 total datapoints and samplerate 44100.0
```
