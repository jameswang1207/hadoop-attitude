# Hadoop HDFS Data Read and Write Operations
## background 
- HDFS has a master and slave kind of architecture. Namenode acts as master and Datanodes as worker. All the metadata information is with namenode and the original data is stored on the datanodes. Keeping all these in mind the below figure will give idea about how data flow happens between the Client interacting with HDFS.

## read a file from HDFS
- step1:First the Client will open the file by giving a call to open() method on FileSystem object, which for HDFS is an instance of DistributedFileSystem class.

- step2:DistributedFileSystem calls the Namenode, using RPC (Remote Procedure Call), to determine the locations of the blocks for the first few blocks of the file. For each block, the namenode returns the addresses of all the datanodes that have a copy of that block. Client will interact with respective datanodes to read the file. Namenode also provide a token to the client which it shows to data node for authentication.

   - The DistributedFileSystem returns an object of FSDataInputStream(an input stream that supports file seeks) to the client for it to read data from FSDataInputStream in turn wraps a DFSInputStream, which manages the datanode and namenode I/O.
   
- step 3: The client then calls read() on the stream. DFSInputStream, which has stored the datanode addresses for the first few blocks in the file, then connects to the first closest datanode for the first block in the file.

- step 4: Data is streamed from the datanode back to the client, which calls read() repeatedly on the stream.

- step 5: When the end of the block is reached, DFSInputStream will close the connection to the datanode, then find the best datanode for the next block. This happens transparently to the client, which from its point of view is just reading a continuous stream.

- step 6: Blocks are read in order, with the DFSInputStream opening new connections to datanodes as the client reads through the stream. It will also call the namnode to retrieve the datanode locations for the next batch of blocks as needed. When the client has finished reading, it calls close() on the FSDataInputStream.
- - ![](./images/HDFS-Read-Operation.png)
