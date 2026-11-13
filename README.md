# MTurk Image and Video Quality Tools

A Flask application for organizing image/video quality studies on
**Amazon Mechanical Turk**. It uploads study media to S3, creates HITs,
collects submitted assignments into MySQL, and provides task-specific
results dashboards.

## Workflow

1. Create an image or video task and enter a description.
2. Upload the media through the web interface.
3. Publish HITs using the corresponding MTurk layout.
4. Refresh submitted results.
5. Inspect ratings and worker-level summaries for each task.

The original code defaults to MTurk's **sandbox**. S3 uploads and other AWS
resources are separate services and can still incur charges.
Do not switch to production without reviewing rewards, assignment settings,
permissions, and the full study workflow.

## Historical runtime

The source contains Python 2 print syntax and requires a compatible
**Python 2.7** environment. The original
[requirements.txt](requirements.txt) pins Flask 1.1.1, boto3 1.9.245,
PyMySQL 0.9.3, and their historical dependencies, including the Python 2
`futures` backport.

Python 2 and this dependency set are obsolete. Treat the project as a
historical prototype, not a current production deployment recipe.

## Configuration checklist

- Create your own MTurk requester setup and image/video layouts from
  [layoutImages.html](layoutImages.html) and [layoutVideo.html](layoutVideo.html).
- Configure layout/type IDs for the legacy `Key.py` interface.
- Provide your own authorized AWS credential configuration. Do not put real
  secrets in tracked source; prefer the standard AWS credential-provider chain
  when adapting the client construction.
- Update hardcoded S3 bucket names, regions, and media URLs to your own resources.
- Configure a local development MySQL database and review its connection settings
  across the source files.
- Make study media accessible only as required by the worker interface.
  Do not broadly disable storage protections or grant unrelated AWS permissions.

The original initialization script creates the `submitted`, `imagefile`,
`videofile`, `metaimage`, and `metavideo` tables in `dbmysql`.
**Running or importing `src/initDB.py` drops and recreates those tables.**
Use only a disposable development database or an explicitly backed-up,
approved initialization target.

## Launching the development interface

The application's working directory must be `src`: its uploads and media
managers use that directory's relative media folders.
Imports mix package-qualified and bare module names, so include the repository
root on `PYTHONPATH`.

After configuring the historical environment and database, a PowerShell launch
from the repository root is:

```powershell
$env:PYTHONPATH = (Get-Location).Path
Set-Location src
python2 app.py
```

Open `http://127.0.0.1:4555/`. The built-in server runs in debug mode; do not
expose it publicly. Upload actions call the HIT-creation routines, so navigating
the application is not an entirely read-only operation.

## Dashboard views

The application includes file/worker rating histories, pairwise comparisons,
rating distributions, submission counts, rating averages, demographic
consistency summaries, age and screen-resolution distributions, and modular
multi-chart layouts. A further view compares file means and standard deviations.

![Example chart entry](src/static/graph1.PNG)

Age/sex inconsistencies are a data-quality signal, not proof that a worker acted
dishonestly. Handle participant identifiers and demographic data under the
study's consent and retention rules.

## Source map

| Path | Purpose |
| --- | --- |
| [src/app.py](src/app.py) | Uploads, task navigation, refresh actions, and dashboard queries |
| [src/createImmHIT.py](src/createImmHIT.py), [src/createVideoHIT.py](src/createVideoHIT.py) | HIT creation |
| [src/imageManager.py](src/imageManager.py), [src/videoManager.py](src/videoManager.py) | S3 media uploads |
| [src/resultsImm.py](src/resultsImm.py), [src/resultsVid.py](src/resultsVid.py) | Submitted-assignment ingestion |
| [src/initDB.py](src/initDB.py) | Destructive schema initialization |
| [src/templates](src/templates) | Task and chart interfaces |

The current image-result collector accepts results after at least one submitted
assignment; this differs from the three-assignment threshold in the earlier
[image/video script project](https://github.com/edoardorex/ImageAndVideoQualityAmazonMTurk).

## Attribution

The original README credits **Edoardo Re, 2019**.
This repository is a fork of
[edoardore/MTurkTools](https://github.com/edoardore/MTurkTools).
Original code and history are preserved; no software license is added or changed.
