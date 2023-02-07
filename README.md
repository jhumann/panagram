# Panagram: Interactive, alignment-free pan-genome browser  

#### Katie Jenike, Sam Kovaka, Shujun Ou, Stephen Hwang, Srividya Ramakrishnan, Ben Langmead, Zach Lippman, Michael Schatz


[An alignment-free pan-genome viewer](https://www.dropbox.com/s/g7snjgr8bs6c2uj/2023.01.17.Panagram.pdf)

Please note: installation instructions and pre-processing scripts are a work in progress. 

# Installation

```
git clone --recursive https://github.com/kjenike/panagram.git
cd panagram
pip install .
```

## Dependencies

Requires python version >=3.7, pip, samtools, and tabix. All other dependencies should be automatically installed via pip.

Panagram relies on [KMC](https://github.com/refresh-bio/KMC) to build its kmer index. This should be installed automatically, however it is possible that the KMC installation will fail but panagram will successfully install. In this case `panagram view` can be run, but `panagram index` will return an error. You may be able to debug the KMC installation by running `make -C KMC py_kmc_api` and attempting to fix any errors, then re-run `pip install -v .` after the errors are fixed.

# Running
Panagram runs in two steps, the pre-processing step (index command) and the viewing (view command). 

# Preprocessing
Usage:
Anchor KMC bitvectors to reference FASTA files to create pan-kmer bitmap
```
usage: panagram index [-h] <config.toml>
```
See example config.toml file for more details on the layout. Must include paths to all of the fasta files and optionally any annotations in gff format. 

Panagram may fail to index datasets with more than 32 genomes. This is **not** a fundamental limitation, and we are working on fixing it.

Currently genome IDs should only contain alphanumeric characters and underscores due to KMC requirements.

# View

Usage:
  Display panagram viewer
```
usage: panagram view [-h] <index_dir/> [genome] [chrom] [start] [end]
  index_dir           Panagram index directory
  genome              Initial anchor genome (optional) 
  chrom               Initial chromosome (optional) 
  start               Initial start coordinate (optional) 
  end                 Initial end coordinate (optional)
  --ndebug            Run server in production mode (important for a public-
                      facing server)
  --port str          Server port (default: 8050)
  --host str          Server address (default: 127.0.0.1)
  --url_base str      A local URL prefix to use app-wide (passed to
                      Dash.dash(url_base_pathname=...)) (default: /)
  --bookmarks str     Bed file with bookmarked regions (default: None)

```

Runs a local Dash server. Browser can be viewed at http://127.0.0.1:8050/ by default.

# Bitdump

Usage:
```
usage: panagram bitdump [-h] [-v bool] index_dir coords step
  Query pan-kmer bitmap generated by "panagram index"/

  index_dir             Panagram index directory
  coords                Coordinates to query in chr:start-end format
  step                  Spacing between output kmers (optimized for multiples
                        of 100) (default: 1)
  -v bool, --verbose bool
                        Output the full bitmap (default: False)
```

# Example run

First download the example_data.zip bacterial data from: 
[http://data.schatz-lab.org/panagram/](http://data.schatz-lab.org/panagram/)

[Direct link](https://bx.bio.jhu.edu/data/panagram/example_data.zip)

Unzip the archive and you will find 5 bacterial genomes plus their annotations
```
unzip example_data.zip
```

To run, first index the genomes:

```
cd example_data
panagram index conf.toml
```

Then you can panagram to visualize (from the example_data directory):
```
panagram view . 
```

From there, you can view the results in your webbrowser at [http://127.0.0.1:8050/](http://127.0.0.1:8050)


# Hosting and Proxies

Panagram uses [Dash](https://dash.plotly.com/introduction) to serve the plotly visualizations. 
By default the dedicated webserver runs on localhost (127.0.0.1) on port 8050, but you can reverse proxy to a different port and path using a web engine 
such as [nginx](https://www.nginx.com/)

For nginx, first reconfigure your nginx configuration file to add (note to be very careful 
with the use of the slash ('/') character):

```
    location /panagram {
      proxy_pass http://127.0.0.1:8050;
    }
```

The retart nginx with 

```
systemctl stop nginx
systemctl start nginx
```

For a secure public-facing server, be sure to run with the option `panagram view --ndebug` to disable debug mode. 

You may also wish to change the base URL path with the `--url_base` option, for example to something like `--url_base /panagram/`. The port and host name can be specified by the `--port` and `--host` options.

Finally you will need to run panagram using `panagram view <dir>`. You will probably want to run this in a loop
in case it needs to be restarted, such as:

```
until panagram view --ndebug .; do echo "restarting"; sleep 1; done
```

We will optimize this process in future releases.

## More information coming soon!
