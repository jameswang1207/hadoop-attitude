# Hadoop HDFS Data Read and Write Operations
## background 
- HDFS has a master and slave kind of architecture. Namenode acts as master and Datanodes as worker. All the metadata information is with namenode and the original data is stored on the datanodes. Keeping all these in mind the below figure will give idea about how data flow happens between the Client interacting with HDFS.

## read a file from HDFS
- step1:First the Client will open the file by giving a call to open() method on FileSystem object, which for HDFS is an instance of DistributedFileSystem class.

- step2:DistributedFileSystem calls the Namenode, using RPC (Remote Procedure Call), to determine the locations of the blocks for the first few blocks of the file. For each block, the namenode returns the addresses of all the datanodes that have a copy of that block. Client will interact with respective datanodes to read the file. Namenode also provide a token to the client which it shows to data node for authentication.

