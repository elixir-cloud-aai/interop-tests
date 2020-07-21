# Running CWL workflows on existing ELIXIR Cloud deployments

## Using cwl-WES

Find available [_cwl_-WES][serv-cwl-wes] deployments
[here][res-elixir-cloud-resources].

### Via CWLab

Coming soon...

### Via WES-cli

Coming soon...

### Via Swagger UI

In your browser, navigate to the Swagger UI route of an existing
WES deployment.

Depending on the service's configuration, you may need to log in with a valid
bearer token (JWT) first. To do so, obtain a valid token from your identity
provider, select the `Authorize` button at the top of the screen, enter the
JWT, prefixed by `Bearer` and separated by a single whitespace in the `value:`
field, and then select `Authorize`.

You can now access the various endpoints by selecting the `Show/Hide`, `List
Operations` or `Expand Operations` fields. Choose an endpoint, if applicable,
provide any required (or desired) parameters, and hit the endpoint's `Try it
out!` button.

### Via `curl`

You can use `curl` to send requests to the cwl-WES API. The following basic
command will send a `GET` request to the `/runs` endpoint of a cwl-WES API
exposed at the URL defined in variable `CWL_WES_API`:

```bash
curl "${CWL_WES_API}/runs"
```

In case the cwl-WES deployment requires authorization, you can add a bearer
token (JWT) `TOKEN` to your request like this:

```bash
curl \
  --header="Authorization: Bearer $TOKEN" \
  "${CWL_WES_API}/runs"
```

You can also add the type of the request ("HTTP verb") explicitly (default:
`GET`):

```bash
curl \
  --request "GET" \
  --header "Authorization: Bearer $TOKEN" \
  "${CWL_WES_API}/runs"
```

Parameters can be passed to the call in different ways, depending on the type
of the parameter.

For _query_ parameters like the paging parameters for the `GET /runs`
endpoint, pass the parameters via the URL. For example, if you want to limit
the response to the last 5 runs, append `?page_size=5` to the URL like so:

```bash
curl \
  --request "GET" \
  --header "Authorization: Bearer $TOKEN" \
  "${CWL_WES_API}/runs?page_size=5"
```

Additional _query_ parameters can be added by separating parameters with an
ampersand (`&`):

```bash
curl \
  --request "GET" \
  --header "Authorization: Bearer $TOKEN" \
  "${CWL_WES_API}/runs?page_size=5&page_token=5ec5419e74d5680009822d11"
```

_Path_ parameters are also passed via the URL, but in contrast to query
parameters, these are passed by replacing the placeholders in the endpoint
definitions. For example, to check on the status for a workflow run with the
identifier `A1B2C3`, you would need to do:

```bash
curl \
  --request "GET" \
  --header "Authorization: Bearer $TOKEN" \
  "${CWL_WES_API}/runs/A1B2C3/status"
```

Finally, _body_ parameters are passed in the payload of the request's body. For
cwl-WES, this applies only to the `POST /runs` workflow. Here, parameters are
passed via the `multipart/form-data` encoding (as dicated by the WES
specification). A basic, valid `curl` call will look something like this:

```bash
curl \
  --request "POST" \
  --header "Authorization: Bearer $TOKEN" \
  --header 'Content-Type: multipart/form-data' \
  --form workflow_params='{"some": "json_object"}' \
  --form workflow_type="CWL" \
  --form worklflow_type="v1.0" \
  --form workflow_url="https://path.to/my_workflow.cwl" \
  "${CWL_WES_API}/runs"
```

## Submitting example workflows

### Hashsplitter workflow

> **TODO**
>  
> * replace reference to workflow URL with one pointing to a specific commit
> * note down cwl-WES and TESK deployments (URLs and image tags/hashes) this
>   was successfully tested with

Required parameters:

* `workflow_params`: `{"input":{"class":"File","path":"ftp://ftp-private.ebi.ac.uk/upload/foivos/test.txt"}}`
* `workflow_type`: `CWL`
* `workflow_version`: `v1.0`
* `workflow_url`: `https://github.com/uniqueg/cwl-example-workflows/blob/master/hashsplitter-workflow.cwl`

Command-line call (`curl`):

```bash
curl \
  --request "POST" \
  --header 'Content-Type: multipart/form-data' \
  --form workflow_params='{"input":{"class":"File","path":"ftp://ftp-private.ebi.ac.uk/upload/foivos/test.txt"}}' \
  --form workflow_type="CWL" \
  --form workflow_type_version="v1.0" \
  --form workflow_url="https://github.com/uniqueg/cwl-example-workflows/blob/master/hashsplitter-workflow.cwl" \
  "${CWL_WES_API}/runs"
```

### MSA workflow

Required parameters:

> **TODO**
>  
> * replace reference to workflow URL with one pointing to a specific commit
> * note down cwl-WES and TESK deployments (URLs and image tags/hashes) this
>   was successfully tested with

* `workflow_params`: `{"fasta_1":{"class":"File","path":"https://github.com/uniqueg/msa_group_compare/blob/master/example_job/data/china_2019_covid_surf_prot_seq.fasta"},"fasta_2":{"class":"File","path":"https://github.com/CompEpigen/msa_group_compare/blob/master/example_job/data/europe_2020_covid_surf_prot_seq.fasta"},"group_name_1":"china_2019","group_name_2":"germany_2020"}`
* `workflow_type`: `CWL`
* `workflow_version`: `v1.0`
* `workflow_url`: `https://github.com/CompEpigen/msa_group_compare/blob/master/CWL/workflows/msa_group_compare.cwl`

Command-line call (`curl`):

```bash
curl \
  --request "POST" \
  --header 'Content-Type: multipart/form-data' \
  --form workflow_params='{"fasta_1":{"class":"File","path":"https://github.com/uniqueg/msa_group_compare/blob/master/example_job/data/china_2019_covid_surf_prot_seq.fasta"},"fasta_2":{"class":"File","path":"https://github.com/CompEpigen/msa_group_compare/blob/master/example_job/data/europe_2020_covid_surf_prot_seq.fasta"},"group_name_1":"china_2019","group_name_2":"germany_2020"}' \
  --form workflow_type="CWL" \
  --form workflow_type_version="v1.0" \
  --form workflow_url="https://github.com/CompEpigen/msa_group_compare/blob/master/CWL/workflows/msa_group_compare.cwl" \
  "${CWL_WES_API}/runs"
```

### RD workflow (full-sized files)

> **TODO**
>  
> * replace reference to workflow URL with one pointing to a specific commit
> * note down cwl-WES and TESK deployments (URLs and image tags/hashes) this
>   was successfully tested with

Required parameters:

* `workflow_params`: `{"chromosome":"1","curl_fastq_urls":{"class":"File","path":"http://86.50.168.37:8000/fastq_files_urls.small.txt"},"curl_known_indels_url":{"class":"File","path":"http://86.50.168.37:8000/known_indels_url.small.txt"},"curl_known_sites_url":{"class":"File","path":"http://86.50.168.37:8000/known_sites_url.small.txt"},"curl_reference_genome_url":{"class":"File","path":"http://86.50.168.37:8000/reference_seq_url.small.txt"},"lftp_out_conf":{"class":"File","path":"http://86.50.168.37:8000/lftp.txt"},"readgroup_str":"@RG\\tID:H947YADXX\\tSM:NA12878\\tPL:ILLUMINA","sample_name":"abc"}`
* `workflow_type`: `CWL`
* `workflow_version`: `v1.0`
* `workflow_url`: `https://github.com/jarnolaitinen/RD_pipeline/blob/master/workflow.cwl`

Command-line call (`curl`):

```bash
curl \
  --request "POST" \
  --header 'Content-Type: multipart/form-data' \
  --form workflow_params='{"chromosome":"1","curl_fastq_urls":{"class":"File","path":"http://86.50.168.37:8000/fastq_files_urls.small.txt"},"curl_known_indels_url":{"class":"File","path":"http://86.50.168.37:8000/known_indels_url.small.txt"},"curl_known_sites_url":{"class":"File","path":"http://86.50.168.37:8000/known_sites_url.small.txt"},"curl_reference_genome_url":{"class":"File","path":"http://86.50.168.37:8000/reference_seq_url.small.txt"},"lftp_out_conf":{"class":"File","path":"http://86.50.168.37:8000/lftp.txt"},"readgroup_str":"@RG\\tID:H947YADXX\\tSM:NA12878\\tPL:ILLUMINA","sample_name":"abc"}' \
  --form workflow_type="CWL" \
  --form workflow_type_version="v1.0" \
  --form workflow_url="https://github.com/jarnolaitinen/RD_pipeline/blob/master/workflow.cwl" \
  "${CWL_WES_API}/runs"
```

### RD workflow (small files)

> **TODO**
>  
> * replace reference to workflow URL with one pointing to a specific commit
> * note down cwl-WES and TESK deployments (URLs and image tags/hashes) this
>   was successfully tested with

Required parameters:

* `workflow_params`: `{"chromosome":"21","curl_fastq_urls":{"class":"File","path":"http://86.50.168.37:8000/fastq_files_urls.small.txt"},"curl_known_indels_url":{"class":"File","path":"http://86.50.168.37:8000/known_indels_url.small.txt"},"curl_known_sites_url":{"class":"File","path":"http://86.50.168.37:8000/known_sites_url.small.txt"},"curl_reference_genome_url":{"class":"File","path":"http://86.50.168.37:8000/reference_seq_url.small.txt"},"lftp_out_conf":{"class":"File","path":"http://86.50.168.37:8000/lftp.txt"},"readgroup_str":"@RG\\tID:H947YADXX\\tSM:NA12878\\tPL:ILLUMINA","sample_name":"abc"}`
* `workflow_type`: `CWL`
* `workflow_version`: `v1.0`
* `workflow_url`: `https://github.com/uniqueg/RD_pipeline/blob/master/workflow.cwl`

Command-line call (`curl`):

```bash
curl \
  --request "POST" \
  --header 'Content-Type: multipart/form-data' \
  --form workflow_params='{"chromosome":"21","curl_fastq_urls":{"class":"File","path":"http://86.50.168.37:8000/fastq_files_urls.small.txt"},"curl_known_indels_url":{"class":"File","path":"http://86.50.168.37:8000/known_indels_url.small.txt"},"curl_known_sites_url":{"class":"File","path":"http://86.50.168.37:8000/known_sites_url.small.txt"},"curl_reference_genome_url":{"class":"File","path":"http://86.50.168.37:8000/reference_seq_url.small.txt"},"lftp_out_conf":{"class":"File","path":"http://86.50.168.37:8000/lftp.txt"},"readgroup_str":"@RG\\tID:H947YADXX\\tSM:NA12878\\tPL:ILLUMINA","sample_name":"abc"}' \
  --form workflow_type="CWL" \
  --form workflow_type_version="v1.0" \
  --form workflow_url="https://github.com/uniqueg/RD_pipeline/blob/master/workflow.cwl" \
  "${CWL_WES_API}/runs"
```

## Miscellaneous

### Stress test cwl-WES

Send `N` workflows in random intervals of between 0 and `T - 1` seconds:

```bash
export N=50 # adjust
export T=6 # adjust

for i in $(seq 1 $N); do
  # your curl command
  sleep $((RANDOM %T))
done
```

[res-elixir-cloud-resources]: <https://github.com/elixir-cloud-aai/elixir-cloud-aai/blob/dev/resources/resources.md>
[serv-cwl-wes]: <https://github.com/elixir-cloud-aai/cwl-WES>