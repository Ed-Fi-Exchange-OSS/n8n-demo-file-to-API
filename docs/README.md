# Evaluation of n8n for Loading CSV Files into the Ed-Fi API

Since 2018, the Ed-Fi Alliance has provided the [Ed-Fi Data
Import](https://github.com/Ed-Fi-Alliance-OSS/Ed-Fi-DataImport) tool as stopgap
for loading CSV files into the Ed-Fi API, covering situations where the source
software vendor has not (yet) built a direct integration into the API.
Ultimately, the Ed-Fi Alliance strongly believes that direct integrations are
the best path forward. When a vendor writes code to push data into an Ed-Fi API,
or provides the API for consumers to pull from, then education organizations can
rest assured that their stored data have met basic referential integrity and
data type validation guarantees.

Moving an entire market is slow work, and some vendors might never write a
direct integration. Yet, the Ed-Fi Alliance does not want to be in the long-term
business of investing in software (Data Import) that enables what we see as an
inferior integration pathway.

In 2023, we commissioned a small study on the state of integration platform as a
service (IPaaS) providers to help us understand the market for alternative
tooling. One platform that looked particularly interesting was
[n8n](https://n8n.io/), which has a paid offering for managing API integrations,
including with files. What we liked about it:

1. It can easily integrate with the API wherever it - or n8n itself - is hosted,
   whether on-prem or in any cloud provider.
2. There is a free self-hosted [community edition](https://docs.n8n.io/hosting/)
   for those willing to manage the service on their own - or, for those who want
   to try the system out locally without needing to create an account first.
3. The source code is available under an [almost open source
   license](https://docs.n8n.io/choose-n8n/faircode-license/#what-source-code-is-covered-by-the-sustainable-use-license),
   should you need modifications for internal use only.
4. Although this is a graphical tool, the workflow files can be exported to JSON
   for source code storage. With the configuration import capability, this also
   means that shared layouts can (theoretically) be shared between
   organizations, much like with Data Import.

The marketing and features look right. Can this tool do the work? We decided to
investigate a few basic requirements through a proof of concept demonstrating:

1. Reading a CSV file.
2. Authenticating to an Ed-Fi API and communicating with it.
3. Branching logic based on record characteristics.
4. Extraction of subsets of data from a complex field.
   * Real-world example: splitting a single "Last Name, First Name" field into
     separate first and last name values.
5. Enrichment of records by looking up additional data in the API.
   * Real-world example: get Student Unique ID from a query.

## Alternatives

There are other excellent integration tools in the marketplace, whether provided
as a cloud-based service or self-hosted by the user. This includes products and
services from specialized Ed-Fi  providers, ETL toolkits from the major cloud
vendors, and other many other worthy third party tools. The goal with this proof
of concept is not to endorse one product over others, but merely to demonstrate
a viable alternative to Data Import, coming from an increasingly competitive
marketplace.

## Proof of Concept

### Walkthrough and Screenshots

See [Ed-Fi Platform as a Service (iPaaS) Proof Of Concept (POC)](poc/README.md)
for a complete write up of the original POC delivery. The CSV and json
(workflow) files mentioned in that document are stored in this repository.

### Running the POC

After delivery of the POC code, we tested locally, running [n8n in
Docker](https://docs.n8n.io/hosting/installation/docker/) and using [live Ed-Fi
demonstration site](https://api.ed-fi.org).

Simplest steps to get started - run the following and then open
http://localhost:5678.

```shell
docker volume create n8n_data

docker run -it --rm --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n
```

Create a sign-in account. Then click the `...` icon in the upper right corner to
import a workflow from a file. There will be errors on the three HTTP Request
nodes. Double click each to resolve the errors and to change the URLs from
`https://localhost:443/` to `https://api.ed-fi.org/v6.1/`. Alternately, we could
have fix this directly in the file before importing, but this is a useful
opportunity to try out the UI. When setting up the OAuth credentials, use these
values:

* Grant type: `Client Credentials`
* Access Token URL: `https://api.ed-fi.org/v6.1/api/oauth/token`
* Client ID: `RvcohKz9zHI4`
* Client Secret: `E1iEFusaNf81xzCxwHfbolkC`
* Authentication: `Header` (both options _should_ work)

Each of the three nodes needs to be reconfigured. Thankfully you can re-use the
credential that was just created. What about the base URL though? Can this be
set by a variable? Unfortunately, variables are only available in the
enterprise plan. So we'll have to paste the correct base URL a couple of times.

The CSV file path is hard-coded. Small problem: the CSV file is on the localhost
computer, not inside of Docker. Click the Save button in the upper right, go
back to the command line and Control-C to exit the running Docker container. Add
the following flag to the Docker command, adjusting the local path as necessary:
`-n c:\path\to\n8n-demo-file-to-api\data:/data`.

Back to the browser: refresh the screen and click the Execute Workflow. If all
goes well, then you'll see a label of "5 items" on the rightmost line, after the
`Merge1` node.

![executed worfklow results](poc/image019.png)

## Conclusion

Although this has not been a full-fledged feature-by-feature comparison, it
appears that n8n could serve as a viable alternative to Data Import and other
tools in many common Ed-Fi data exchange scenarios.

The POC demonstrates reading a file from the local filesystem; applying code to
manipulate records; and performing additional lookups to further enrich the
records before POSTing them to the Ed-Fi API. n8n also has other built-in tools
that allow a non-programmer to perform common tasks, such as downloading a file
from an FTP server, filtering out records, or looking for differences between
datasets. The POC also shows how easy it is to import and export workflows for
sharing between different organizations.

The lowest friction operational experience will come with the paid plans. An
organization running in a self-hosted mode would need to explore the command
line interface in more detail, and would want to study the [logging
capabilities](https://docs.n8n.io/hosting/logging-monitoring/). The lack of
variable support will require a little work, but this should not be much of a
challenge to the data engineer(s): one might store a token in the source file,
and orchestrate a step to substitute in the proper setting during execution.
