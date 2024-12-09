# Volara Satya Proof of Contribution

This repository provides proof of contribution using Satya validators for the Volara dataset on the Vana network.

## Overview

This template provides a basic structure for building proof tasks that:

1. Read input files from the `/input` directory.
2. Process the data securely, running any necessary validations to prove the data authentic, unique, high quality, etc.
3. Write proof results to the `/output/results.json` file in the following format:

```json
{
  "dlp_id": 19, // DLP ID is found in the Root Network contract after the DLP is registered
  "valid": false, // A single boolean to summarize if the file is considered valid in this DLP
  "score": 0.7614457831325301, // A score between 0 and 1 for the file, used to determine how valuable the file is. This can be an aggregation of the individual scores below.
  "authenticity": 1.0, // A score between 0 and 1 to rate if the file has been tampered with
  "ownership": 1.0, // A score between 0 and 1 to verify the ownership of the file
  "quality": 0.6024096385542169, // A score between 0 and 1 to show the quality of the file
  "uniqueness": 0, // A score between 0 and 1 to show unique the file is, compared to others in the DLP
  "attributes": {
    // Custom attributes that can be added to the proof to provide extra context about the encrypted file
    "total_score": 0.5,
    "score_threshold": 0.83,
    "email_verified": true
  }
}
```

The project is designed to work with [Gramine](https://gramine.readthedocs.io/en/latest/), a lightweight library OS that enables running unmodified applications in secure enclaves, such as Intel SGX (Software Guard Extensions). This allows the code to run in a trusted execution environment, ensuring confidentiality and integrity of the computation.

## Project Structure

- `volara_proof/`: Contains the main proof logic
  - `proof.py`: Implements the proof generation logic
  - `__main__.py`: Entry point for the proof execution
- `demo/`: Contains sample input and output for testing
- `.github/workflows/`: CI/CD pipeline for building and releasing
- `Dockerfile`: Defines the container image for the proof task
- `volara-proof.manifest.template`: Gramine manifest template for running securely in an Intel SGX enclave
- `config.yaml`: Configuration file for Gramine Shielded Containers (GSC)

## Getting Started

To use this template:

1. Fork this repository
2. Modify the `volara_proof/proof.py` file to implement your specific proof logic
3. Update the `volara-proof.manifest.template` if you need to add any additional files or change the configuration
4. Commit your changes and push to your repository

## Customizing the Proof Logic

The main proof logic is implemented in `volara_proof/proof.py`. To customize it, update the `Proof.generate()` function to change how input files are processed.

The proof can be configured using environment variables. When running in an enclave, the environment variables must be defined in the `volara-proof.manifest.template` file as well. The following environment variables are used for this demo proof:

- `COOKIES`: The cookies for the data contributor

## Local Development

To run the proof locally, without Gramine, you can use Docker:

```
docker build -t volara-proof .
docker run \
--rm \
--volume $(pwd)/demo/sealed:/sealed \
--volume $(pwd)/demo/input:/input \
--volume $(pwd)/demo/output:/output \
--env USER_EMAIL=user123@gmail.com \
volara-proof
```

## Building and Releasing

This template includes a GitHub Actions workflow that automatically:

1. Builds a Docker image with your code
2. Creates a Gramine-shielded container (GSC) image
3. Publishes the GSC image as a GitHub release

**Important:** To use this workflow, you must generate a signing key and add it to your GitHub secrets. Follow these steps:

1. Generate a signing key (see instructions below)
2. Add the key as a GitHub secret named `SIGNING_KEY`
3. Push your changes to the `main` branch or create a pull request

### Generating the Gramine Signing Key (Required)

Before building and signing your graminized Docker image, you must generate a signing key. This key is crucial for creating secure SGX enclaves. Here's how to generate it:

1. If you have Gramine installed:

   ```
   gramine-sgx-gen-private-key enclave-key.pem
   ```

2. If you don't have Gramine, use OpenSSL:

   ```
   openssl genrsa -3 -out enclave-key.pem 3072
   ```

After generating the key:

1. Keep this key secure, as it will be used to sign your enclaves.
2. Add the contents of `enclave-key.pem` as a GitHub secret named `SIGNING_KEY`.

This key is essential for the `gsc sign-image` step in the GSC workflow.

## Running with SGX

Intel SGX (Software Guard Extensions) is a set of security-related instruction codes built into modern Intel CPUs. It allows parts of a program to be executed in a secure enclave, isolated from the rest of the system.

To load a released image with docker, copy the URL from the release and run:

```
curl -L https://address/of/gsc-volara-proof.tar.gz | docker load
```

To run the image:

```
docker run \
--rm \
--volume /gsc-volara-proof/input:/input \
--volume /gsc-volara-proof/output:/output \
--device /dev/sgx_enclave:/dev/sgx_enclave \
--volume /var/run/aesmd:/var/run/aesmd \
--volume /mnt/gsc-volara-proof/sealed:/sealed \
--env USER_EMAIL=user123@gmail.com \
gsc-volara-proof
```

Remember to populate the `/input` directory with the files you want to process.

## Security Features

This template leverages several security features:

1. **Secure Enclaves**: The proof runs inside an SGX enclave, isolating it from the rest of the system.
2. **Encrypted Storage**: The `/sealed` directory is automatically encrypted/decrypted by Gramine, providing secure storage for sensitive data.
3. **Input/Output Isolation**: Input and output directories are mounted separately, ensuring clear data flow boundaries.
4. **Minimal Attack Surface**: The Gramine manifest limits the files and resources accessible to the enclave, reducing potential vulnerabilities.

## Contributing

If you have suggestions for improving this template, please open an issue or submit a pull request.

## License

[MIT License](LICENSE)

