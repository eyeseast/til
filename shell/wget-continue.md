# Resuming failed downloads with WGET

I'm currently, as I type this, downloading a 110 GB map file [for the whole planet](https://maps.protomaps.com/builds/). That takes long enough that the download has failed when I've stepped away and my laptop has gone to sleep.

Thankfully, `wget` has a way to pick up where I left off, using the `--continue` (or `-c`) flag (found [here](https://www.cyberciti.biz/tips/wget-resume-broken-download.html)).

```sh
# start a download
wget https://build.protomaps.com/20240104.pmtiles

# hours later, it fails
# resume where we left off
wget -c https://build.protomaps.com/20240104.pmtiles
```

Now if I can just figure out resumable uploads.
