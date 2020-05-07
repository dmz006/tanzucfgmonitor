# tanzucfgmonitor
Ruby script to monitor for opsman config changes

Tanzu Config Monitor will parse opsman configuration files and monitor specific variables for changes

```
Usage:
       tanzucfgmonitor [options]

Example:
       tanzuconfigmonitor -f TAS_config.yaml -p "cf-config,bosh-director-config"

Process Config Options:
       pivotal-container-service-config
       bosh-director-config
       cf-config

where [options] are:
  -f, --configfile=<s>      Config file to monitor
  -m, --monitorlist=<s>     File with list of variables to monitor (default: tanzu-monitor.yaml)
  -d, --databasefile=<s>    What file to save the results in (default: tanzucfgmonitor.db)
  -p, --process=<s>         Process configs (default: pivotal-container-service-config,bosh-director-config)
  -q, --query               Query database values
  -v, --version             Print version and exit
  -h, --help                Show this message
```

# To Install
gem install yaml optimist sqlite3

# Operation
The repo includes a sample config file saved from opsman that includes a few variables that are worth monitoring for BOSH, TKGI (formerly Pivotal Container Service PKS), and TAS (Formerly Pivotal Application Service PAS).

You must specify a config file and you can use multiple configs over multiple runs.

The script has 2 modes of operation: process and query.  It defaults to process with the process type for TKGI and BOSH unless you specify cf-config or another valid config.  If a query is run, it will dump the current value for all variables in the database (based on the configs specified in the process option).

Example processing run:
`./tanzucfgmonitor -f TAS_config.yaml -m tanzu-monitor.yaml -p "cf-config,bosh-director,config" -d tas.db`

The script works by reading the list of variables to monitor.  This defaults to tanzu-monitor.yaml and has the default suggested variables to start with for BOSH, TKGI, or TAS.

The script then reads in the config file specified and gets the values of the variables from the monitorlist.  

If the variable value is unknown (1st time read) it will save the value in the SQLite database (tanzucfgmonitor.db) and output that the value was added and set the script exit code to be non-zero (1).

If the variable is known and it is different in the database from the last run the script will update the database, output to the screen the change and set the script exit code to be non-zero (1).

If the variable is known and hasn't changed nothing will be displayed and the exit code will stay as 0.

This script can be used in-line with the platform pipeline.  Just have the pipeline save the current tile configurations and process with tanzucfgmonitor script (sample OM/BOSH output to be added soon).

If the script exits with a non-zero exit code of 1 ($? == 1) the script can then use whatever mechanism the organization uses to create a ticket or incident to investigate the changes identified.

To save config files use om-cli and save the config
BOSH Director Config 
`echo "bosh-director-config:" > BOSH_config.yaml && om -e env.yml -k staged-director-config | sed 's|^\(.*\)$|  \1|' >> BOSH_config.yaml`

TKGI Config 
`echo "pivotal-container-service-config:" > TKGI_config.yaml && om -e env.yml -k staged-config --product-name pivotal-container-service | sed 's|^\(.*\)$|  \1|' >> TKGI_config.yaml`

TAS Config 
`echo "cf-config:" > TAS_config.yaml && om -e env.yml -k staged-config --product-name cf | sed 's|^\(.*\)$|  \1|' >> TAS_config.yaml`

# Notice
This script only works on the tile configurations, not on the genenerated manifest.
