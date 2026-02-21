---
title: Paperless cloud backups
url: /paperless-cloud-backup
date:
categories:
  - homelab
tags:
build:
  list: never
  render: always
---

I store all of my important (and many not so important) documents in my
[Paperless-ngx](https://paperless-ngx.com/) instance running on my homelab.
Given this importance, I wanted to add an additional backup of my documents and
the metadata. Having them in cloud storage, while not ideal for privacy reasons,
would also give me a way to access the documents easily if I am unable to
directly for whatever reason.
<!--more-->

I chose to use Dropbox for this, because I have a lot of space available from
sending invites years ago, and I have it synced onto my laptop which then gives
me an additional backup there too.

</br>

My initial instinct on how to solve this was to just run the dropbox daemon on
my homelab and create a symlink to wherever paperless stores my documents. I
quickly found that this wouldn't work for a couple of reasons:
 1. The dropbox daemon doesn't run on Arm, and my homelab is a Raspberry Pi.
 2. Paperless doesn't store metadata in a way that's easy to backup.

To solve 1., I spent sometime looking into dropbox sync scripts that do work on
the Pi, and then writing my own script to do the same thing, but I wasn't happy
with any of these. It didn't take too much of this research before I was
reminded of the existance of [Rclone](https://rclone.org/), which is much more
robust than any of these scripts and allows me to do a one way sync, which is
ideal for my backup. It does bring a new problem to solve, which is that Rclone
doesn't run as a daemon, it needs to be triggered somehow to sync.

The solution to 2. is the [Paperless document exporter
script](https://docs.paperless-ngx.com/administration/#exporter). This is
Paperless' recommended backup solution, it does provide all of the metadata and
there's an equivalent import script should the backup need to be restored.
Perfect, almost. This has the same issue as Rclone, in that it needs to be
triggered somehow.

So now we have two scripts that need to be run in sequence, frequently enough
such that my backup is kept sufficiently up to date. And I guess now is the time
to mention that I'm running Paperless in docker and intend to keep using the
supplied image, so the backup should happen in my docker compose file.

## Scheduling the backup

A simple timed schedule is a classic way to run backups, so that's what I went
with initially. Using the official Rclone docker image and
[Ofelia](https://github.com/mcuadros/ofelia) to trigger the document export and
then Rclone sync. Below is an abbreviated docker compose file with hourly
backups added. The full version is available
[here](https://gist.github.com/sams96/dc97a06fcb669465d0ca44d4041b2cad), but if
you plan to use this I recommend basing yours on [a more up to date compose file
from
Paperless](https://github.com/paperless-ngx/paperless-ngx/tree/main/docker/compose).

```yaml
version: "3.4"
services:
  broker: ...
  db: ...

  webserver:
    ...
    volumes:
      - export:/usr/src/paperless/export
      ...
    labels:
      ofelia.enabled: "true"
      ofelia.job-exec.export-job.schedule: "@hourly"
      ofelia.job-exec.export-job.command: "document_exporter --no-thumbnail ../export"

  rclone:
    container_name: rclone
    image: rclone/rclone:latest
    volumes:
      - export:/data
      - ./rclone/config:/root/.config/rclone # Place Rclone config here
    labels:
      ofelia.enabled: "true"
    command: "copy /data [remote:location]"

  ofelia:
    image: mcuadros/ofelia:latest
    restart: unless-stopped
    depends_on:
      - webserver
    command: daemon --docker
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    labels:
      # backup job is run 5 minutes after the hour to give the export time to run
      ofelia.job-run.backup.schedule: "0 5 * * * *"
      ofelia.job-run.backup.container: "rclone"

volumes:
  export:
```

Rather than figure out running Rclone once the document exporter has
finished, I just set it to run 5 minutes later, which works well enough. I've
been using this for the past ~6 months with no issues.

However, I don't actually update documents in Paperless that often, maybe once
or twice a week, so it feels unnecessary to run the backups every hour but then
I don't want to leave documents un-backed up for too long. Ideally the backups
could be triggered when I actually do something in Paperless.

## Backup on change

Paperless provides the ability to send webhooks on document addition and update,
as part of their [workflows
feature](https://docs.paperless-ngx.com/usage/#workflows), so my next idea is
using this to trigger the document export and sync job instead of doing it
hourly. To do this I created [float](https://github.com/sams96/float). The idea
of float is essentially to provide similar functionality to Ofelia, but
triggered by webhooks instead of on a schedule.
