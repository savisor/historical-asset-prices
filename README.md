# Historical Asset Prices

A comprehensive collection of historical forex price data stored in Parquet format, covering major currency pairs across multiple timeframes.

Perfect for backtesting trading strategies, conducting market analysis, and financial research. Data is readily accessible and can be queried using modern analytics tools like DuckDB, ClickHouse, Polars, and DataFusion.

## Downloading All Pairs Data

Download all pairs data to a destination directory using curl. Replace `/path/to/destination` with your desired path:

```bash
DESTINATION="~/Downloads/historical-asset-prices"
timeframes=("1M" "5M" "15M" "1H" "4H" "1D" "1W")

# Add or remove pairs as needed
for pair in AUDUSD EURJPY EURUSD GBPJPY GBPUSD NZDUSD USDCHF USDJPY; do
  mkdir -p "$DESTINATION/$pair"
  for tf in "${timeframes[@]}"; do
    curl -L -o "$DESTINATION/$pair/$tf.parquet" \
      "https://raw.githubusercontent.com/savisor/historical-asset-prices/main/$pair/$tf.parquet"
  done
done
```

## Querying Data with DuckDB in Docker

Run DuckDB in a Docker container with a single command. Replace `/path/to/data` with your data directory:

```bash
# On macOS, it's common to store datasets in your user's Documents or Downloads folder.
# For example, using the Downloads folder:
docker run -it -v ~/Downloads/historical-asset-prices:/data ubuntu:22.04 bash -c "cd /data && apt-get update -qq && apt-get install -y wget unzip > /dev/null && wget -q https://github.com/duckdb/duckdb/releases/latest/download/duckdb_cli-linux-amd64.zip && unzip -q duckdb_cli-linux-amd64.zip && chmod +x duckdb && ./duckdb"
```

Inside DuckDB, enable httpfs extension and query parquet files:

```sql
INSTALL httpfs;
LOAD httpfs;

-- Query local parquet file
SELECT * FROM read_parquet('/data/EURUSD/1D.parquet') LIMIT 10;

-- Query remote parquet file directly
SELECT * FROM read_parquet('https://raw.githubusercontent.com/savisor/historical-asset-prices/main/EURUSD/1D.parquet') LIMIT 10;
```

## Querying Data with ClickHouse in Docker

Run ClickHouse in a Docker container with a single command:

```bash
docker run -it -v ~/Downloads/historical-asset-prices:/data clickhouse/clickhouse-server clickhouse local
```

Inside ClickHouse, query parquet files:

```sql
-- Query local parquet file
SELECT * FROM file('/data/EURUSD/1D.parquet', Parquet) LIMIT 10;

-- Query remote parquet file directly
SELECT * FROM url('https://raw.githubusercontent.com/savisor/historical-asset-prices/main/EURUSD/1D.parquet', Parquet) LIMIT 10;
```

## Querying Data with Polars in Docker

Run Polars in a Docker container with a single command:

```bash
docker run -it -v ~/Downloads/historical-asset-prices:/data python:3.11-slim bash -c "pip install -q polars pyarrow requests && cd /data && python"
```

Inside Python, query parquet files:

```python
import polars as pl

# Query local parquet file
df = pl.read_parquet('/data/EURUSD/1D.parquet')
print(df.head(10))

# Query remote parquet file directly
df = pl.read_parquet('https://raw.githubusercontent.com/savisor/historical-asset-prices/main/EURUSD/1D.parquet')
print(df.head(10))
```
