# Parquet Plugin for HPCC-Systems

The Parquet Plugin for HPCC-Systems is a powerful tool designed to facilitate the fast transfer of data stored in a columnar format to the ECL (Enterprise Control Language) data format. This plugin provides seamless integration between Parquet files and HPCC-Systems, enabling efficient data processing and analysis.

## Installation

The plugin uses vcpkg and can be installed by creating a separate build directory from the platform and running the following commands:
```
cd ./parquet-build
cmake -DPARQUETEMBED=ON ../HPCC-Platform
make -j4 package
sudo dpkg -i ./hpccsystems-plugin-parquetembed_<version>.deb
```

## Documentation

[Doxygen](https://www.doxygen.nl/index.html) can be used to create nice HTML documentation for the code. Call/caller graphs are also generated for functions if you have [dot](https://www.graphviz.org/download/) installed and available on your path.

Assuming `doxygen` is on your path, you can build the documentation via:
```
cd plugins/parquet
doxygen Doxyfile
```

## Features

The Parquet Plugin offers the following main functions:

### Regular Files

#### 1. Reading Parquet Files

The Read function allows ECL programmers to create an ECL dataset from both regular and partitioned Parquet files. It leverages the Apache Arrow interface for Parquet to efficiently stream data from ECL to the plugin, ensuring optimized data transfer.

In order to read a Parquet file it is necessary to define the record structure of the file you intend to read with the names of the fields as stored in the Parquet file and the type that you wish to read them as. It is possible for a Parquet file to have field names that contain characters that are incompatible with the ECL field name definition. For example, ECL field names are case insensitive causing an issue when trying to read Parquet fields with uppercase letters. To read field names of this type an XPATH can be passed as seen in the following example:

```
layout := RECORD
    STRING name;
    STRING job_id {XPATH('jobID')};
END;

dataset := ParquetIO.Read(layout, '/source/directory/data.parquet');
```

#### 2. Writing Parquet Files

The Write function empowers ECL programmers to write ECL datasets to Parquet files. By leveraging the Parquet format's columnar storage capabilities, this function provides efficient compression and optimized storage for data. There is an optional argument that sets the overwrite behavior of the plugin. The default value is false meaning it will throw an error if the target file already exists.

The Parquet Plugin supports all available Arrow compression types. Specifying the compression when writing is optional and defaults to Uncompressed. The options for compressing your files are Snappy, GZip, Brotli, LZ4, LZ4Frame, LZ4Hadoop, ZSTD, Uncompressed.

```
overwriteOption := TRUE;
compressionOption := 'Snappy';

ParquetIO.Write(inDataset, '/output/directory/data.parquet', overwriteOption, compressionOption);
```

### Partitioned Files (Tabular Datasets)

#### 1. Reading Partitioned Files

The Read Partition function extends the Read functionality by enabling ECL programmers to read from partitioned Parquet files.

```
github_dataset := ParquetIO.ReadPartition(layout, '/source/directory/partioned_dataset');
```

#### 2. Writing Partitioned Files

For partitioning parquet files all you need to do is run the Write function on Thor rather than hthor and each worker will create its own parquet file.
