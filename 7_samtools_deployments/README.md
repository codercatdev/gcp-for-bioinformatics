# GCP Cloud Compute Service Options for Bioinformatics

Genomic researchers frequently use open source data analysis tools.  **Samtools** is a one such example. This toolset is defined as *"Utilities for the analysis of genomic files which use the Sequence Alignment/Map (SAM) format"*. Samtools source code can be found at this [link](http://www.htslib.org/doc/samtools.html).

This page will detail possible compute solutions on GCP.  The purpose is to learn | compare various GCP options for data analysis using compute services via a straightforward bioinformatics example. See the GCP Sketchnote below from [@pvergadia](https://twitter.com/pvergadia) and [link to full sketchnote page](https://thecloudgirl.dev/gcpcompute.html) to get a high-level overview of compute service option types available on GCP.

<img src="https://thecloudgirl.dev/images/vs.jpg" width=900>

## What is it

**Samtools** is a suite of open source programs for interacting with high-throughput sequencing data. Samtools is a set of utilities that manipulate alignments in the SAM (Sequence Alignment/Map), BAM, and CRAM formats. It converts between the formats, does sorting, merging and indexing, and can retrieve reads in any regions swiftly. Samtools is designed to work on a stream
It consists of three separate repositories:

- **Samtools**: Reading/writing/editing/indexing/viewing SAM/BAM/CRAM format
- **BCFtools**: Reading/writing BCF2/VCF/gVCF files and calling/filtering/summarising SNP and short indel sequence variants
- **HTSlib**: A C library for reading/writing high-throughput sequencing data

NOTE: Samtools and BCFtools both use HTSlib internally, but these source packages contain their own copies of htslib so they can be built independently.

One or more of the available Samtools is often used in a genomic data anlysis pipeline.  An example pipeline which shows the use of samtools, linked from an article posted by [UT in Austin, TX](https://wikis.utexas.edu/display/CoreNGSTools/The+Basic+Alignment+Workflow), is shown below.

<img src="https://wikis.utexas.edu/download/attachments/66697008/image2016-5-23%2018%3A17%3A24.png?version=1&modificationDate=1464027446000&api=v2" width=900>

---

## Prepare to Test

There are a number of steps to prepare before testing.  These include downloading and installing samtools.  Also you'll need to get sample input data. Your test results will differ, depending on your input data type, size and values.  As a starting point, we will use the `samtools index` command for our testing.  An example (using the GCP Life Sciences API service) is shown below.  This example uses a public container with `samtools` and also a BAM file from GCP public datasets.  Use of these public artifacts makes initial testing simpler.

```
gcloud beta lifesciences pipelines run \
    --regions us-east1 \
    --command-line 'samtools index ${BAM} ${BAI}' \
    --docker-image "gcr.io/cloud-lifesciences/samtools" \
    --inputs BAM=gs://genomics-public-data/NA12878.chr20.sample.bam \
    --outputs BAI=gs://mayo-demos-life-sciences/NA12878.chr20.sample.bam.bai
```

### Samtools: Get it / Set it up

For other testing scenarios, we'll need to get the binaries for `samtools`.  Directions/links are below.  

- Download Samtools (open source and free) - download [here](http://www.htslib.org/)
- Set up Samtools - directions [here](http://www.htslib.org/download/) --OR--
	- Use GCR samtools container - [here](https://console.cloud.google.com/gcr/images/cloud-lifesciences/GLOBAL/samtools)

### Sample Input Data: Get it / Set it up

As we continue to test, we'll need to set up storage and get more input files.  We'll use a Cloud Storage bucket and also data stored in public datasets on GCP.

- Create one or more Cloud Storage buckets in your GCP account
	- Add your input data to your bucket
- Use public datasets on GCP
	- BAM input file in public GCP bucket - at `gs://genomics-public-data/NA12878.chr20.sample.bam`
	- TCGA public data in BQ dataset - [here](https://cloud.google.com/life-sciences/docs/resources/public-datasets/tcga#datasets)
- (Optional) Upload Samatools source files to your GCP Cloud Storage bucket
- (Optional) Index/Tag buckets / BigQuery datasets as a DataMesh using GCP DataPlex services

----

## Perform The Tests

This page will cover a series of types of `samtools` task pipeline architectures on a number of GCP Services.  Most examples will use Cloud Storage buckets for input/output files. Shown below is a reference architecture which shows multiple options on GCP.

- Single task (or group) as a pipeline, using a script
- Containerized pipeline - use a public container (from gcr.io) vs build a container
- Using different GCP compute services - raw VM, other

<img src="https://github.com/lynnlangit/gcp-for-bioinformatics/blob/master/images/option-arch.png" width=900>

## Possible Run locations

Can be run on compute IaaS, SaaS or PaaS services on the Google Cloud Platform.

### IaaS - run on VM

- **Compute Engine/VM** - run as script 
- **Compute Engine/VM** - run as container 
	- from DockerHub - at this URL: https://hub.docker.com/r/biocontainers/samtools/ 
		- using command `docker pull biocontainers/samtools`
	- from GCR - at this URL: gcr.io/cloud-lifesciences/samtools 
		- using command `docker pull gcr.io/cloud-lifesciences/samtools:latest`

### SaaS - run via services

These are cloud service endpoints, compute services which are pre-configured for various scenarios (batch, machine learning, etc...). Like VMs these servcies are billed by the invocation or by the time interval (minutes or seconds).

- **Colabs Notebook** - run as Jupyter Notebook on pre-optimized GCE VM instance
	- example link (from 2014) - [link](https://colab.research.google.com/github/BenLangmead/comp-genomics-class/blob/master/notebooks/SAM.ipynb)
- **Cloud Function** - trigger script from file upload to bucket using `lambda-type` serverless compute function
- **CloudRun Job** - run container on file in bucket using serverless compute job
- **Google Batch** - run cluster of VMs from compute trigger (CloudRun or Cloud Function) - `samtools` example - [here](https://github.com/lynnlangit/gcp-for-bioinformatics/blob/master/2_Virtual_Machines_&_Docker_Containers/6b_Use_Batch_API.md#how-to-do-this---samtools-example)
	- example `job.json` file (below) uses the `samtools index` command with a public samtools container, a public bam file (from/to your bucket pattern)
	- NOTE: replace the listed Cloud Storage bucket name and path (volume | gcs | remotePath value) with your own bucket name and your path

```
{
    "taskGroups": [
        {
            "taskSpec": {
                "runnables": [
                    {
                        "container": {
                            "imageUri":"gcr.io/cloud-lifesciences/samtools",
                            "entrypoint": "/bin/sh",
                            "commands": [
                                "-c",    
                                "samtools index ${BAM} /mnt/disks/share/${BAI}"            
                            ]
                        },
                        "environment": {
                            "variables": {
                                "BAM": "gs://genomics-public-data/NA12878.chr20.sample.bam",
                                "BAI": "NA12878.chr20.sample.bam.bai"                             
                            }
                        }
                    }
                ],
                    "volumes": [
                        {
                            "gcs": {
                                "remotePath": "batch-demo-lynn/samtools-lynn/"
                            },
                            "mountPath": "/mnt/disks/share"
                        }
                    ],
                "computeResource": {
                    "cpuMilli": 2000,
                    "memoryMib": 2000
                },
                "maxRetryCount": 3,
                "maxRunDuration": "100000s"
            },
            "taskCount": 1,
            "parallelism": 10
        }
    ],
    "logsPolicy":{
        "destination": "CLOUD_LOGGING"
    }
}
```
#### Batch gcloud commands

```
gcloud batch jobs submit JOB_NAME \
  --location LOCATION \
  --config <configFile>.json

gcloud batch jobs describe JOB_NAME \ 
   --location LOCATION 

gcloud batch jobs list

gcloud batch tasks list \ 
   --location LOCATION \
   --job JOB_NAME

gcloud batch tasks describe TASK_INDEX \
  --location LOCATION \
  --job JOB_NAME \
  --task_group TASK_GROUP_NAME

```

### PaaS - run on managed VMs/containers

- **Vertex AI (Jupyter) Notebook instance** - Puython, TensorFlow, R and more
	- use with R language and|or when visualizing results
	- Example use 'Rsamtools` library - [link](http://www.bioconductor.org/packages/2.14/bioc/html/Rsamtools.html)
- **GKE/Google Kubernetes Engine**  - run container cluster
	- create custom cluster, requires extensive manual configuration and ongoing manual management
	- allows for custom security, scalability and control

---

## Monitor and Scale

- Compute Engine - type | size of VM, add NLB and monitoring
- Functions/ CloudRun - monitor and adjust configuration
- Kubernetes Engine - set up, monitor and scale

## Use Bioinformatics Workflow Language - Nextflow

Test by running using the Nextflow workflow language and engine on GCP (w/Google Batch)   
- Use `nf-core` version of samtools. [Nf-core](https://nf-co.re/) is a registry of Nextflow data analysis pipelines.
	- Get the NF modules/pipeline code from `nf-core` registry - [link](https://nf-co.re/modules/samtools_view) 
	- Get the NF module source code from GitHub - [link](https://github.com/nf-modules/samtools)
- Example Nextflow pipeline that uses `samtools` - pipeline name is HGTSEQ - [link](https://nf-co.re/hgtseq). Pipeline diagram (subway map) for this pipeline is shown below. Diagram is from nf-core documentation.

<img src="https://raw.githubusercontent.com/nf-core/hgtseq/1.0.0/docs/images/hgtseq_pipeline_metromap.png" width=900>

## Links

- :hammer: Workflow example `fastq to bam` - [link](http://www.htslib.org/workflow/fastq.html)
- :octocat: GitHub source repo for `samtools` - [link](https://github.com/samtools/samtools)
- 🗃️ Google Cloud Life Sciences public container images - [link](https://console.cloud.google.com/gcr/images/cloud-lifesciences)
- :book: Article '10 Years of Samtools' - [link](http://gigasciencejournal.com/blog/play-it-again-samtools/)



