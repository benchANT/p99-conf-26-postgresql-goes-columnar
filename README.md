# PostgreSQL Goes Columnar: How Far Can It Really Push Analytics?

Companion data for the talk **"PostgreSQL Goes Columnar: How Far Can It Really Push Analytics?"** by Daniel Seybold (Co-Founder, [benchANT](https://benchant.com)) at [P99 CONF 2026](https://p99conf.io/).

The talk benchmarks PostgreSQL's analytical extensions (pg_duckdb, Citus Columnar) against plain PostgreSQL and a specialized OLAP engine (ClickHouse) on TPC-DS and ClickBench. This repository contains the slides and the raw benchmark results behind them.

## Repository structure

```
.
├── slides/                      Presentation deck (PDF)
├── results/
│   ├── clickbench/
│   │   ├── load/                Data loading runs
│   │   └── query/               Query runs (43 ClickBench queries)
│   └── tpc-ds-original/
│       ├── 10gb/                TPC-DS, scale factor 10 GB
│       └── 100gb/               TPC-DS, scale factor 100 GB
└── LICENSE                      CC BY-SA 4.0
```

Each leaf folder under `results/` is **one benchmark run**, named `olap_<benchmark>_<configuration>_<n>_<runId>`. The configuration in the name tells you which system was tested:

| Configuration | System under test |
|---|---|
| `postgresql-plain` | Vanilla PostgreSQL 17 (row store) |
| `postgresql-pgduckdb` / `pg_duckdb` | PostgreSQL 17 with the pg_duckdb extension |
| `postgresql-citus` | PostgreSQL 17 with Citus Columnar |
| `clickhouse-aws` | ClickHouse Cloud on AWS, as a dedicated OLAP reference |
| `*-indexed-*` | Same as above, with additional indexes (ClickBench only) |

`load` runs ingest the dataset; `query` runs execute the queries against the loaded data (see `parentRunId` in a run's `evaluationScenario.json` for the load run it builds on).

Not every configuration was run for every benchmark and scale factor. The folders in `results/` are the complete list.

## Contents of a run folder

| File | Description |
|---|---|
| `0_run.txt` | Benchmark client output with per-query latency metrics (average, min, max, 95th/99th percentile) and return status |
| `*-load_*.log` | Data load log (`clickbench-load`, `tpcds-load`) |
| `evaluationScenario.json` | Full scenario definition: DBMS, version, instance types, PostgreSQL settings, benchmark parameters |
| `postgresql.conf`, `postgresql.version` | PostgreSQL configuration and version actually used |
| `benchANT_versions` | Versions of the benchANT platform components used for the run |
| `*_hardware_facts*.json`, `*_resources.json` | Hardware facts and provisioned cloud resources for the DB and benchmark nodes |
| `timestamp.json` | Run start/end timestamps |
| `monitoring.xlsx`, `raw_metrics/*.csv` | System metrics (CPU, memory, disk, network, ...) collected during the run, where available |
| `airflowTaskInstanceDetails.json` | Orchestration task metadata |
| `dbaas_*.json`, `dbaas_metrics.txt` | Service details and metrics for the managed ClickHouse Cloud deployment |

## Test setup (summary)

- **Cloud:** AWS EC2, `eu-central-1`
- **PostgreSQL node:** `m7i.2xlarge`, Ubuntu 22.04, 500 GB gp3 volume (12,000 IOPS), PostgreSQL 17
- **Benchmark client node:** `c6i.4xlarge`
- **ClickHouse:** ClickHouse Cloud (AWS `eu-central-1`, 2 replicas with 16 GB memory each, version 25.12)
- **Workload:** single client thread, queries executed sequentially

The authoritative details for each run are in its `evaluationScenario.json`, `postgresql.conf` and `*_hardware_facts*.json`.

## Reproducibility notes

- The runs were executed and orchestrated with the benchANT benchmarking platform. Results here are the raw, unprocessed outputs; the charts in the slides are derived from them.
- Credentials and account identifiers in the raw files were replaced with `REDACTED`. All benchmark infrastructure was torn down after the runs, so the IP addresses and instance IDs in the hardware facts no longer point to anything.
- Each configuration was run for a single iteration, so results are indicative rather than a statistically rigorous comparison.
- Cloud pricing, instance availability and extension versions change. Re-running on different hardware or versions may give different numbers.

## License

Slides and data are licensed under [Creative Commons Attribution-ShareAlike 4.0 International](LICENSE) (CC BY-SA 4.0).

## Contact

Questions about the data or the methodology: [benchANT](https://benchant.com) / Daniel Seybold.
