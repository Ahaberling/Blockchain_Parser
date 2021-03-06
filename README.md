# Bitcoin to Neo4j

The code in this repository is an end-to-end solution for loading all bitcoin transactions in the blockchain into a
Neo4j graph database. It is written in Python 3 and optimized for Linux, but the single-core version also works on MacOS and Windows. Windows
users might have to compile some of the dependencies manually.

## Considerations
When running this script, you will essentially turn your computer into a _full node_ of the bitcoin network. This means
that you will have to **download the ENTIRE (!) blockchain** (> 230 GB) on your PC. For the import and the database
to run smoothly, I strongly recommend using a system fulfilling the following minimum requirements:

* 2 TB of disk space (1 TB for raw data, HDD sufficient + 1 TB of fast SSD space for the databases)
* As much RAM as you can afford and fit on your Mainboard. The faster your disk, the less RAM you will need,
 but I recommend a minimum of 32 GB.
* Intel Core i5/AMD Ryzen 5 @ 3,5 GHz or better
* Unmetered broadband internet access, ideally >= 50 Mbit/s

**Depending on your bandwidth and your system configuration, this script might take between 4-10 days to complete. During
the last phase of the import process (Neo4j import) you will not be able to use your computer!**

## Instructions

**1. Install Neo4j**
  * On Debian/Ubuntu:  `sudo apt-get install neo4j`
  * All others click [here](https://neo4j.com/docs/operations-manual/current/installation/) for Instructions

**1.1 Install APOC for Neo4J**

Find instructions [here](https://github.com/neo4j-contrib/neo4j-apoc-procedures#manual-installation-download-latest-release)

**2. Install RocksDB**

Click [here](https://python-rocksdb.readthedocs.io/en/latest/installation.html) for instructions.

**3. Install Bitcoin Core**
  * [Linux](https://bitcoin.org/en/full-node#linux-instructions) (Install Daemon only)
  * [MacOS](https://bitcoin.org/en/full-node#mac-os-x-instructions)
  * [Windows](https://bitcoin.org/en/full-node#windows-instructions)

**4. Configure the bitcoin client**

Prior to running your bitcoin node for the first time, you should adjust the bitcoin.conf file as follows:

```
txindex=1
server=1
```
Typical locations for the bitcoin.conf file are listed below. If you do not find a bitcon.conf file in this location,
you can create a new one using any text editor.
```
Windows:        C:\Users\<username>\AppData\Roaming\Bitcoin                                             
Linux:          /home/<username>/.bitcoin/bitcoin.conf                                                           
MacOS:          /Users/<username>/Library/Application Support/Bitcoin/bitcoin.conf
```
**5. Run the Bitcoin client**
  * Linux/MacOS: `bitcoind -daemon`
  * Windows: Open command prompt and tpye `C:\Program Files\Bitcoin\daemon\bitcoind`

 Your system is now downloading the ledger. Once this is complete, you should stop the client before proceeding.
 This can be done with `bitcoin-cli -stop`

 **6. Install the parsing library**

 As of now, the library used for parsing the Blockchain is not available through pip. To install it manually follow
 the instructions on the [project page](https://github.com/SchaeferJ/python-bitcoin-blockchain-parser).


**7. Clone this Repo**

Download or clone this repository, save it in the place of your choice and install the dependencies by running

`pip install -r requirements.txt`

 **7.1 Linux only: Update ulimits**

 The data processing and especially RocksDB will require A LOT of file operations. Linux often restricts the number of
 file handles that can be acquired by a process.

 * Type `ulimit -n` to display your current limit. If a low limit is set
 (typically 1024) wou will most likely run into problems when parsing the blockchain.

 * Open the system.conf file: `sudo nano /etc/systemd/system.conf`

 * Search for the line `#DefaultLimitNOFILE=`, remove the # and set the limit to 1000000

 * Reboot and type `ulimit -n` to check if change was successful

**8. Run the CSV generator**

Open a terminal and enter:
`python3 btc_parallel.py`

If you are using a weak machine or run this code on a system that is not Linux, you should use the single-threaded version:
`python3 btc-to-csv.py`

The following flags are available:
```
--help:             Displays a help message
--startblock        Block height to start at. Defaults to 0 (Genesis)
--endblock          Block height to stop at. Defaults to -1 (entire chain). **Must be specified
                    when using parallel processing**
--btcdir            Directory of Bitcoin Core. Defaults to system standard
--outdir            Output directory. Defaults to working directory.
--dbdir             Directory for RocksDB database. Defaults to working directory.
--mem               Maximum allocated memory
--cores             Maximum allocated cores (multiprocessing only)
```

**9. Import CSVs to Neo4j**

* Linux: `bash ./csv-to-neo4j.sh`
* Others: Please create appropriate [header files](https://neo4j.com/docs/operations-manual/current/tools/import/file-header-format/) and refer to the documentation of the [import tool](https://neo4j.com/docs/operations-manual/current/tools/import/)

**10. Construct additional attributes**

For efficiency reasons, not all node attributes are generated during parsing. In this last step, you have
to generate the attributes manually:

* Linux/MacOS: 'neo4j-shell -file feature-construction.cql'
* Windows: 'Neo4jShell.bat -file feature-construction.cql'
