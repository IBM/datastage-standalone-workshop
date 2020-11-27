# Admin Guide

This section has hints and tips for instructors:

## Provisioning environments

Use <https://bluedemos.com/show/2543> to provision instances of the Skytap environment as needed.

* Only IBMers can provision an instance, you'll need to log in with your w3ID.
* You're limited to one request at a time.
* You can schedule how long you want to keep the instance live.
* You'll be emailed a link and a password. Distribute these to the lab attendees.
* There's no cost to running these instances.
* Instances will automatically suspend after 2 hours of inactivity. Just turn them back on to continue.

## Enable the Operations Console

Access the `iis-server` VM.

Log in with the credentials above. Run the following commands.

Change to the Information Server install path:

```bash
$ cd /opt/IBM/InformationServer/Server/DSODB
```

Edit the `DSODBConfig.cfg` file. set the `DSODBON` value to `1`. It should be the first value at the top of the file. Save your changes and exit your editor.

```ini
DSODBON=1
```

If you're not familiar with `vi` you can use `sed`

```bash
sed -i 's/DSODBON=0/DSODBON=1/g' DSODBConfig.cfg
```

Restart your console:

```bash
$ cd /opt/IBM/InformationServer/Server/DSODB/bin
$ ./DSAppWatcher.sh -start
```

## Copy the data over

Access the `iis-server` VM.

Log in with the credentials above. Run the following commands.

Change to the right folder:

```bash
$ cd /IBMdemos
```

Download the data:

> **NOTE**: the command uses `-O` (not zero)

```bash
$ wget http://ibm.biz/datastage-standalone-data -O raviga-products.csv
```

Verify the content

```bash
$ cat raviga-products.csv
RAV8XLIGA,90033
...
RAV9XLIGA,90033
```
