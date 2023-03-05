conntrack collector shim for [node-exporter](https://github.com/prometheus/node_exporter)
===

# Overview

In recent kernels (like Ubuntu 22.04) `NF_CONNTRACK_PROCFS` is obsolete and disabled.
This prevents conntrack collector from scraping statistics

See https://github.com/prometheus/node_exporter/issues/2491

Symptom is collector silently doesn't work, despite being enabled

```
node_scrape_collector_success{collector="conntrack"} 0
```

This shim provides an almost complete substitute via textfile collector

# Install 

Shim relies on `conntrack(8)` being available, which is a recommended approach to gathering stats.
You also will need Python 3

1. Install conntrack

```
sudo apt-get -yyq update
sudo apt-get -yyq install python3 conntrack
```

2. Install the shim

```
curl -sLo /usr/local/bin/node-exporter-conntrack-shim \
  https://raw.githubusercontent.com/noroutine/node-exporter-conntrack-shim/master/node-exporter-conntrack-shim
chmod +x /usr/local/bin/node-exporter-conntrack-shim
```

3. Add a cronjob to generate metrics textfile into proper textfiler directory (adjust as needed)

```
* * * * * root /usr/local/bin/node-exporter-conntrack-shim > /var/lib/node_exporter/node-exporter-conntrack-shim.prom
```

For example

```
cat <<EOF > /etc/cron.d/node-exporter-conntrack-shim
* * * * * root /usr/local/bin/node-exporter-conntrack-shim > /var/lib/node_exporter/node-exporter-conntrack-shim.prom
EOF
```

4. Make sure node-exporter has textfile collector enabled by adding below arguments

```
  --collector.textfile --collector.textfile.directory=/var/lib/node_exporter
```

# Limitations

`node_nf_conntrack_entries_limit` and `node_nf_conntrack_stat_ignore` metrics from collector are missing, since `conntrack(8)` is not providing this information