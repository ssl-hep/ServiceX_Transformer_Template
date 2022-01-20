# ServiceX Transformer Template

![](https://github.com/ssl-hep/ServiceX_Transformer_Template/workflows/Docker%20Hub/badge.svg)

Apply this template to a new repository to create a transformer instance for 
ServiceX

## Customize the Template
In your instantiation of this template, you need to customize the following to
turn this into a runnable transformer:
1. Update the image name in the `.github/workflows/buildpush.yaml` steps
2. Update the image name throughout this README
3. Update the repo name for accessing the build status badge above
4. Add your real code to the `transform_single_file` function in transformer.py

## Usage

You can invoke the transformer from the command line. For example:

```
> docker run --rm -it sslhep/servicex_transformer_template:latest python transformer.py --help
usage: transformer.py [-h]
                      [--tree TREE] [--attrs ATTR_NAMES]
                      [--path PATH] [--limit LIMIT]
                      [--result-destination {object-store,output-dir}]
                      [--output-dir OUTPUT_DIR]
                      [--result-format {arrow,parquet,root-file}]
                      [--rabbit-uri RABBIT_URI] [--request-id REQUEST_ID]


optional arguments:
  -h, --help            show this help message and exit
  --tree TREE           Tree from which columns will be inspected
  --attrs ATTR_NAMES    List of attributes to extract
  --path PATH           Path to single Root file to transform
  --limit LIMIT         Max number of events to process
  --result-destination {object-store,output-dir}
                        object-store
  --output-dir OUTPUT_DIR
                        Local directory to output results
  --result-format {arrow,parquet,root-file}
                        arrow, parquet, root-file
  --rabbit-uri RABBIT_URI
  --request-id REQUEST_ID
                        Request ID to read from queue
```

You will need an X509 proxy avaiable as a mountable volume. The X509 Secret
container can do using your credentials and cert:
```bash
docker run --rm \
    --mount type=bind,source=$HOME/.globus,readonly,target=/etc/grid-certs \
    --mount type=bind,source="$(pwd)"/secrets/secrets.txt,target=/servicex/secrets.txt \
    --mount type=volume,source=x509,target=/etc/grid-security \
    --name=x509-secrets sslhep/x509-secrets:latest
```


## Development
```bash
 python3 -m pip install -r requirements.txt
```

### Testing

#### Testing by running against known file and C++ files

Under tests you'll find input files needed to try this out. Use the following docker conmmand.

```
docker run --rm -it \
  --mount type=volume,source=x509,target=/etc/grid-security-ro \
  --mount type=bind,source=$(pwd),target=/code \
  --mount type=bind,source=$(pwd)/tests,target=/data \
  --mount type=bind,source=$(pwd)/tests/r21_test_cpp_files,target=/generated,readonly \
  sslhep/servicex_transformer_template:latest bash
```

Then use `trasformer.py` and pass it the `--path` argument.

For example:
```bash
 python transformer.py --path /data/jets_10_events.root --result-format root-file --output-dir /tmp --result-destination output-dir
```