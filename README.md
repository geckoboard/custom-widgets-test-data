# Custom widget test data

Sample data feeds for testing Geckoboard [Custom Widgets](https://developer-custom.geckoboard.com/).
Each file is a payload that a polling widget (one configured with a URL data
feed) can fetch over HTTPS.

To use one, put its raw URL in the widget's URL data feed:

```
https://raw.githubusercontent.com/geckoboard/custom-widgets-test-data/main/pie-chart/all-colours.json
```

That URL is pinned to `main` rather than to a commit, so editing the file on
`main` changes what the widget receives without touching the widget config.

Polling needs this repo to stay public: Geckoboard fetches the URL
unauthenticated. The same payloads work for push widgets, covered in
[Push widgets](#push-widgets) below.

## Layout

One folder per widget type, named as the developer docs name the type. Every type
in the docs has at least one sample.

## Widget types

### Bar Chart (`bar-chart/`)

- `basic.json` — Seven-year series, y axis formatted as USD currency.

### Bullet Graph (`bullet-graph/`)

- `basic.json` — Horizontal bullet with red, amber and green bands, a projected measure, and a comparative marker at 600.

### Funnel (`funnel/`)

- `eight-steps.json` — Eight descending steps.

### Gauge (`gauge/`)

- `small-range.json` — Value 23 in a 10 to 30 range.
- `large-range.json` — Value 1788 in a 10 to 9889 range.

### Highcharts (`highcharts/`)

Split a level further, by the format each payload is written in, because that is
what its variations turn on. The widget accepts a chart config in six shapes: two
file formats (a JavaScript object literal, or strict JSON) crossed with three
levels of wrapping (none, a `highchart` key holding a string, or a `highchart` key
holding an object). Each shape takes a different path through the parser, so each
gets its own file.

- `js/bare-object.js` — Object literal, evaluated as the config. Not valid JSON: unquoted keys and a trailing comma.
- `js/highchart-key-string.js` — Object literal with a `highchart` key whose string value is evaluated as the config.
- `js/highchart-key-object.js` — Object literal with a `highchart` key whose object value is the config.
- `json/bare-object.json` — Strict JSON, used as the config.
- `json/highchart-key-string.json` — JSON with a `highchart` key whose string value is evaluated as the config.
- `json/highchart-key-object.json` — JSON with a `highchart` key whose object value is the config.
- `js/non-ascii-titles-and-series-names.js` — Chart title, axis title and series names in Polish, Chinese, Arabic, Japanese and Danish. Every string is multi-byte UTF-8, and the Arabic one is right-to-left.

To make a new variation, add a file here and use its raw URL. There is a
[Highcharts getting-started tutorial](https://www.highcharts.com/docs/getting-started/your-first-chart)
if you need a config to start from.

### Leaderboard (`leaderboard/`)

- `basic.json` — Fourteen items, six of them carrying `previous_rank` and eight without, so both the moved and unmoved states render.

### Line Chart (`line-chart/`)

- `single-series-currency.json` — One unlabelled series, y axis formatted as USD currency.
- `two-series-with-labels.json` — Two series with month labels on the x axis and no y axis format.

### List (`list/`)

- `basic.json` — A top-level array rather than an object, which is this widget's payload shape. Seven items, one with a coloured label, and long descriptions mixed with short ones.

### Map (`map/`)

- `all-point-formats.json` — Every way to place a point: city name, city with region code, latitude and longitude, hostname, and IP address. Sizes and colours on some points only.

### Monitoring (`monitoring/`)

- `up.json` — Status `Up`.
- `down.json` — Status `Down`.

### Number & Secondary Stat (`number-and-secondary-stat/`)

- `basic.json` — One value with a label.
- `prefix.json` — One value with a `€` prefix.
- `reverse.json` — Two values falling from 565 to 559, with `"type": "reverse"` to invert how that change is treated.
- `comparison.json` — Two values, so the widget shows the percentage change.
- `trendline.json` — One value plus seven points for the trendline.
- `absolute.json` — Two values with `"absolute": true`, so the change shows as a number instead of a percentage.
- `absolute.xml` — The same payload in XML. The only XML sample here.

### Pie Chart (`pie-chart/`)

- `all-colours.json` — Four slices, each with a colour.
- `partial-colours.json` — The same slices with a colour on the first one only, so the rest fall back to defaults.

### RAG (`rag/`)

- `three-values.json` — Three values with labels.

### Text (`text/`)

- `two-items.json` — Two items, one of each `type`.
- `long-text.json` — Two long paragraphs, for overflow behaviour.

## Push widgets

A push widget takes the same payload as a polling one, wrapped in an envelope
carrying your API key, so the JSON files here work for both. The widget key comes
from the widget's own config, and the API key from your account page.

Push accepts JSON only. `number-and-secondary-stat/absolute.xml` and the
files under `highcharts/js/` cannot be pushed, as they are XML and JavaScript.

To push one payload, replacing the two keys:

```bash
curl -X POST https://push.geckoboard.com/v1/send/WIDGET_KEY \
  -H 'Content-Type: application/json' \
  -d '{"api_key":"API_KEY","data":{"item":[{"value":5723,"text":"Total paying customers"}]}}'
```

To push any file in this repo without cloning it:

```bash
WIDGET_KEY=your-widget-key
API_KEY=your-api-key
FILE=leaderboard/basic.json

curl -s "https://raw.githubusercontent.com/geckoboard/custom-widgets-test-data/main/$FILE" \
  | python3 -c 'import json,sys;print(json.dumps({"api_key":sys.argv[1],"data":json.load(sys.stdin)}))' "$API_KEY" \
  | curl -X POST "https://push.geckoboard.com/v1/send/$WIDGET_KEY" \
      -H 'Content-Type: application/json' --data @-
```

Or from a clone, swapping `$FILE` for whichever payload you want:

```bash
python3 -c 'import json,sys;print(json.dumps({"api_key":sys.argv[1],"data":json.load(open(sys.argv[2]))}))' \
  "$API_KEY" "$FILE" \
  | curl -X POST "https://push.geckoboard.com/v1/send/$WIDGET_KEY" \
      -H 'Content-Type: application/json' --data @-
```

A widget that rejects the payload still returns `200`, so check the widget on the
dashboard rather than the response. The
[developer docs](https://developer-custom.geckoboard.com/#push-overview) cover the
payload size and rate limits.
