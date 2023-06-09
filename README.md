# Extended DDoS Dissector
### (For Usage with the [Reassembler](https://github.com/j0nezz/reassembler))
This GitHub repository entails the source code and files developed as part of the Master's Thesis at the University of Zurich in 2022/2023.
It is based on a fork of the [original DDoS Dissector](https://github.com/ddos-clearing-house/ddos_dissector).
As such, the original README has been extended to reflect the proposed changes.
While the changes are backwards compatible, please referr to the [original DDoS Dissector](https://github.com/ddos-clearing-house/ddos_dissector) repository if you intend to use the Dissector without the Reassembler.

--- 
_This part has been copied and adapted_
## DDoS Dissector

The Dissector summarizes DDoS attack traffic from stored traffic captures (pcap/flows). The resulting summary is in the
form of a DDoS Fingerprint; a JSON file in which the attack's characteristics are described.

## How to use the Dissector

### Option 1: in Docker

You can run DDoS Dissector in a docker container. This way, you do not have to install dependencies yourself and can
start analyzing traffic captures right away. The only requirement is to
have [Docker](https://docs.docker.com/get-docker/) installed and running.

1. Pull the docker image from [docker hub](https://hub.docker.com/r/ddosclearinghouse/dissector): `docker pull ddosclearinghouse/dissector`
2. Run dissector in a docker container:
    ```bash
    docker run -i --network="host" \
    --mount type=bind,source=/abs-path/to/config.ini,target=/etc/config.ini \
    -v /abs-path/to/data:/data \
    ddosclearinghouse/dissector -f /data/capture_file [options]
    ```
   **Note:** We bind-mount the [config file](config.ini.example) with DDoS-DB and MISP tokens to `/etc/config.ini`, and create a volume mount for the location of capture files.
   We use the local network to also allow connections to a locally running instance of DDoS-DB or MISP. Fingerprints are saved in `your-data-volume/fingerprints`


### Option 2: Installed locally

1. Install the dependencies to read PCAPs (tshark) and Flows (nfdump):

    1. https://tshark.dev/
    2. https://github.com/phaag/nfdump

2. Clone the Dissector repository

    ```bash
    git clone https://github.com/ddos-clearing-house/ddos_dissector;
    cd ddos_dissector;
    ```

3. [Advised] create a python virtual environment or conda environment for the dissector and install the python requirements:

    Venv:
    ```bash
    python -m venv ./python-venv
    source python-venv/bin/activate
    pip install -r requirements.txt
    ```
    [Conda](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html):
    ```bash
    conda create -n dissector python=3.10
    conda activate dissector
    conda install pip
    pip install -r requirements.txt
    ```

4. Get a traffic capture file to be analized (PCAP files should have the `.pcap` extension, Flows should have the `.nfdump` extension)

5. Run the dissector:
    ```bash
    python src/main.py -f data/attack_traffic.nfdump --summary
    ```
## Options

```
    ____  _                     __
   / __ \(_)____________  _____/ /_____  _____
  / / / / / ___/ ___/ _ \/ ___/ __/ __ \/ ___/
 / /_/ / (__  |__  )  __/ /__/ /_/ /_/ / /
/_____/_/____/____/\___/\___/\__/\____/_/

usage: main.py [-h] -f FILES [FILES ...] [--summary] [--output OUTPUT] [--config CONFIG] [--nprocesses N] [--target TARGETS [TARGETS ...]] [--ddosdb]
               [--misp] [--noverify] [--debug] [--show-target] [-n NR_FP] [-t THRESHOLD] [-e] [-l LOCATION]

options:
  -h, --help            show this help message and exit
  -f FILES [FILES ...], --file FILES [FILES ...]
                        Path to Flow / PCAP file(s)
  --summary             Optional: print fingerprint without source addresses
  --output OUTPUT       Path to directory in which to save the fingerprint (default ./fingerprints)
  --config CONFIG       Path to DDoS-DB and/or MISP config file (default /etc/config.ini)
  --nprocesses N        Number of processes used to concurrently read PCAPs (default is the number of CPU cores)
  --target TARGETS [TARGETS ...]
                        Optional: target IP address or subnet of this attack
  --ddosdb              Optional: directly upload fingerprint to DDoS-DB
  --misp                Optional: directly upload fingerprint to MISP
  --noverify            Optional: Don't verify TLS certificates
  --debug               Optional: show debug messages
  --show-target         Optional: Do NOT anonymize the target IP address / network in the fingerprint
  -n NR_FP, --nfingerprints NR_FP
                        Optional: Generate fingerprints for top n targeted IPs
  -t THRESHOLD, --threshold THRESHOLD
                        Optional: Detection threshold for attack target selection
  -e, --extended-format
                        Optional: Use the extended Fingerprint Format
  -l LOCATION, --location LOCATION
                        Optional: Recording location (used in extended FP format)

Example: python src/main.py -f /data/part1.nfdump /data/part2.nfdump --summary -e -l 178.x.x.x
```

## DDoS Fingerprint format

### [Click here](fingerprint_format.md)

## Example Fingerprints using the Extended Format

**Note: numbers and addresses are fabricated but are inspired by real fingerprints.**

<details>
   <summary>(Click to expand) Fingerprint from PCAP data: DNS amplification attack with fragmented packets</summary>

```json
{
    "attack_vectors": [
        {
            "service": null,
            "protocol": "TCP",
            "fraction_of_attack": 1.0,
            "source_port": "random",
            "destination_ports": "random",
            "tcp_flags": {
                "......S.": 0.997
            },
            "nr_packets": 2093500,
            "nr_megabytes": 127,
            "time_start": "2018-11-03T15:28:00.776482+00:00",
            "duration_seconds": 22067,
            "source_ips": [
                "172.16.0.5",
                "172.217.6.234",
                "52.85.90.206",
                "172.217.11.10",
                ...
            ],
            "ethernet_type": {
                "IPv4": 1.0
            },
            "frame_len": {
                "60": 0.998
            },
            "fragmentation_offset": {
                "0": 1.0
            },
            "ttl": {
                "243": 0.997
            },
            "ttl_by_source": {
                "34.208.7.98": [
                    230
                ],
                "172.16.0.5": [
                    45,
                    240,
                    241,
                    231,
                    51,
                    48,
                    ...
                ],
                "172.217.3.106": [
                    116,
                    121,
                    53,
                    58
                ],
                "172.217.6.234": [
                    53,
                    58
                ],
                "172.217.9.234": [
                    53,
                    58
                ],
                "172.217.10.74": [
                    53,
                    58
                ],
                "172.217.11.10": [
                    53,
                    58
                ]
            },
            "nr_packets_by_source": {
                "52.85.90.206": 440,
                "54.149.111.157": 3,
                "54.186.113.55": 24,
                "72.21.91.29": 3,
                "91.189.92.20": 3,
                "172.16.0.5": 2092545,
                "172.217.3.106": 90,
                ...
            }
        }
    ],
    "target": "192.168.50.4",
    "tags": [
        "TCP",
        "TCP flood attack",
        "TCP SYN flag attack"
    ],
    "key": "60764fd912f7bd9f047ca885f652758f",
    "location": "192.168.50.4",
    "time_start": "2018-11-03T15:28:00.776482+00:00",
    "time_end": "2018-11-03T21:35:48.097688+00:00",
    "duration_seconds": 22067,
    "total_packets": 2093500,
    "total_megabytes": 127,
    "total_ips": 25,
    "avg_bps": 46066,
    "avg_pps": 94,
    "avg_Bpp": 60
}
```

</details>