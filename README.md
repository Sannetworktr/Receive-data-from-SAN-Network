# Receive data from SAN Network

The API serves recent observations from community stations that have consented to sharing. Credit SAN Network when using the data for non-commercial research.

## 1. Create a read key

Open [Member Panel → Receive data](https://sannetwork.net/member?lang=en#api) and create a read key for the service you need. Use a separate key for each service and pass it in the `X-API-Key` HTTP header. Copy and store the full key securely when it is shown at creation.

## 2. Make your first request

```bash
# ADS-B — bounded geographic area
curl -sS --fail-with-body \
  'https://sannetwork.net/api/v1/flights/live?bbox=30,35,35,40&limit=25' \
  -H "X-API-Key: $SAN_ADSB_READ_KEY"

# AIS — one vessel
curl -sS --fail-with-body \
  'https://sannetwork.net/api/v1/maritime/vessels?mmsi=271000001&limit=1' \
  -H "X-API-Key: $SAN_AIS_READ_KEY"

# VHF — list available MP3 segments
curl -sS --fail-with-body 'https://sannetwork.net/api/v1/vhf/records?limit=10' \
  -H "X-API-Key: $SAN_VHF_READ_KEY"
```

For VHF, resolve the relative `audio_url` returned by the list against `https://sannetwork.net` and request it with the same VHF read key. The response content type is `audio/mpeg`.

### Query filters

| Parameter | Use |
| --- | --- |
| `bbox` | West longitude, south latitude, east longitude, north latitude. Maximum span of 20° on each axis. |
| `hex` | Six-character ICAO address for ADS-B. |
| `mmsi` | Nine-digit MMSI for AIS. |
| `station_id` | Restricts results to a known station that has consented to sharing. |
| `limit` | 1–100 records. |
| `fields` | Comma-separated list of up to 15 supported fields, for example `hex,lat,lon,timestamp`. |

Use `bbox` or `hex` for ADS-B and `bbox` or `mmsi` for AIS. JSON responses include `data`, `count`, `truncated`, and `timestamp`. A successful response with `count: 0` means there were no fresh matching observations at the time of the request.

## Quotas and errors

Default member quotas are 30 requests per minute, 2,000 requests per day, 100 MiB per day, and 1,024 MiB per month. Quotas are shared across keys. Daily and monthly counters use UTC. Check your panel for your assigned limits. Bulk downloads, pagination, and archive exports are prohibited.

| Response | What to do |
| --- | --- |
| `401` | Check that the key is current and active. |
| `403` | Check the service scope and IP allowlist. |
| `404` | Check the request path and whether the VHF segment is still available. |
| `422` | Correct the query filters or record limit. |
| `429` | Honor `Retry-After` and reduce request frequency. |
| `503` | Retry with increasing delays. |
