# Running WDL with Cromwell using TESK with FTP

## Prerequisites
1. A running TESK instance in FTP-mode.
2. An FTP service available to TESK and to use for Cromwell. In case you are using vSFTP, uncomment the following two lines in /etc/vsftpd.conf or else TESK will not be able to run the tasks submitted by Cromwell:
```bash
ascii_upload_enable=YES
ascii_download_enable=YES
```
3. A machine to run Cromwell, with the Java Runtime Environment (JRE) installed.

## Deploy Cromwell
### Download Cromwell jar
Downlad the Cromwell jar file for the version required from here:
```
https://github.com/broadinstitute/cromwell/releases
```
### Configuration
Create a new file and add the content below (add your own configuration values where appropriate)
```bash
filesystems.ftp.global.config 
{
    # Maximum number of connections that will be established per user per host. This is across the entire Cromwell instance
    max-connection-per-server-per-user = 30
    # FTP connection port to use
    connection-port: 21
    # FTP connection mode
    connection-mode = "passive"
  }

engine.filesystems.ftp 
{
    username = "Your FTP username"
    password = "Your FTP password"
}

backend 
{
    default = "TES"
    providers 
    {
        TES
        {
            actor-factory = "cromwell.backend.impl.tes.TesBackendLifecycleActorFactory"
            config 
            {
                # FTP Root ftp path where all output files will be stored
                root = "Your FTP remote storage location" # e.g: ftp://ftp.yourserver.com/cromwell-executions/
                # URL of the TESK server endpoint
                endpoint = "Your TESK endpoint url" #e.g. "https://tesk-location.com/v1/tasks"
                concurrent-job-limit = 10
                # FTP authentication
                filesystems.ftp.auth 
                {
                    username = "Your FTP username"
                    password = "Your FTP password"
                }
                #location inside the TESK pods that the TESK PVC will be mounted on
                dockerRoot = "/cromwell-executions"
                glob-link-command = "ls -L GLOB_PATTERN 2> /dev/null | xargs -I ? ln -s ? GLOB_DIRECTORY"
            }
        }
    }
}
```

### Deploying Cromwell
Use the following command to deploy Cromwell:
```bash
java -Dconfig.file=<cromwell_config_file_location> -jar cromwell-<version>.jar server
```
and wait a few seconds for the server to come up.

## Testing Cromwell
The Cromwell server should now be running at <machine_id_address>:8000. If you see the swagger API page, then you can use the [Cromwell REST API](https://cromwell.readthedocs.io/en/stable/api/RESTAPI/) to run a workflow.