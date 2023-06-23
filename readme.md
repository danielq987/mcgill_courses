# Hooray

Ran this script to fetch from each of the 527 pages and extract all the relevant course page links to a single file `raw_links`.
```bash
#!/bin/bash

SCRIPT_DIR=$( cd -- "$( dirname -- "${BASH_SOURCE[0]}" )" &> /dev/null && pwd )

find_string='views-field-field-course-title-long'

# Fetch from each page
for i in {0..526}; do
  # Status message
  echo "fetching from page $i"

  # curl -> fetch data from page
  # egrep/grep -> find the relevant links on the page
  # and then write to a file called ./raw/meow
  curl -s "https://www.mcgill.ca/study/2023-2024/courses/search?page=$i" \
    | egrep $find_string \
    | grep -Po '(?<=href=\")[^\"]+'\
     >> $SCRIPT_DIR/raw_links
done
```

Ran this convoluted line to transform each link in `raw_links` into a proper url and into the correct format for curl (the tool which fetches the pages).
Writes to `curl_config`.
```bash
sed -e 's/^/url=\"https:\/\/mcgill.ca/' raw_links | sed -e 's/$/\"/' | sed -e '/.*/{p;s/.*\//output=\"test\//g;}' > curl_config
```

And then this line fetches all the pages using the config file `curl_config` and dumps each html page into a separate file. Parallel mode because otherwise it would take forever.
```bash
curl --parallel --parallel-immediate --parallel-max 50 --config curl_config --location
```

End result is each course having its own html file - further HTML parsing can be done with Python and BS4.
