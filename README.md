## What it do
Search for something on Yelp, you will get a URL with all of your search paramaters (`...yelp.com/search?...`).
This will start there, grab all of the links to the individual business pages on Yelp (which I get through the API) and then...

#### (default)
search take your keywords, and add them to your search query one at a time. For example, if you want to find a donut shop that caters to a lot of food allergies, you would search yelp for "donuts", configure whatever filters you want, grab the url, and run `cargo run -- --url="<url>" --keywords="soy free","gluten free","dairy free"` and it will search "donuts soy free", "donuts gluten free", "donuts dairy free" and keeps count of how many times businesses show up in the search results. The more keywords, the more descriptive your data will be.

#### If `--crawl`
go to those pages and search the reviews for different keywords. It will count how many reviews pop up for each keyword. *I know* that this isn't a perfect count but yelp does not make it easy to analyze its data, so you can't customize the query at all. It will search for reviews that contain your whole query, but it will look for different forms of a word (i.e. single/plurals). If I *could* do one query that `OR`s my keywords, I *would*. But as it stands, I can't, so this has to make *a lot* of HTTP requests.

## Motivation
Combing through Yelp to find friendly businesses to partner with for fundraising events is a pain in the ass. Also, trying to learn Rust.

## Bottlenecks
Make your time tradeoffs wisely when choosing your initial query and keywords:
* Yelp's maximum limit for business searches is 50, max 1000 businesses are returned -> up to 20 requests to just get the business links

If `--crawl`:
* Each business page gets a request -> up to 1000
* Yelp review queries: as far as I can tell, Yelp doesn't support anything more than basic `contains` queries. So I can't search for reviews that contain `fundraise OR charity`. Have to do each individually -> <num_businesses> x <num_keywords> requests
* I have to rate limit
Else:
* If you searched "cheap food" businesses with the `--keywords=fundraise,"non profit",charity", then this will search Yelp for "cheap food fundraise", cheap food non profit", and "ceap food charity". Pagination rules still apply -> <num_keywords> x (<num_businesses> / 50) requests


# WARNING about using `--crawl`
Yelp will block your IP address if you crawl their website, so I put the thread to sleep for .5 seconds every time it searches through the reviews for a keyword


## Directions
```
USAGE:
    yelp_scraper [FLAGS] [OPTIONS] --url <URL>

FLAGS:
    -c, --crawl      Run the much longer, web crawler function to return # of reviews with your keywords per businesses.
                     Defaults to false
    -h, --help       Prints help information
    -V, --version    Prints version information

OPTIONS:
    -k, --keywords <KEYWORDS>...    Keywords to search for
    -o, --output <OUTPUT PATH>      Where you want the output file to go
    -u, --url <URL>                 The url you get when you make a search on yelp. Please wrap in quotation marks
```

The URL is **required**. The other arguments and their defaults are listed below. *Note*: URL has to be wrapped in quotation marks and if you want multiword keywords, wrap it in quotation marks (e.g. `--keywords=fundraise,"bob ross",charity`)

```
cargo run -- --url="<some yelp url you get when you search for something>" --keywords=fundraise --out=./out.txt
```

If you want to use the slow, web scraper version:
```
cargo run -- --url="<some yelp url you get when you search for something>" --crawl
```

<hr>

### v0.0.1 MVP:
- [x] comb through all of the index links for a given query
- [x] scrape yelp reviews
- [x] print to txt file


### v0.1.0 MVP:
- [x] accept arguments at run time: initial url, output path, keywords


### v1.0.0 MVP:
- [ ] make sure the Yelp platforms query parameters map to the api query parameters
    - [x] term
    - [x] location: neighborhoods, lat/lon/radius
    - [x] categories
    - [x] price range
    - [ ] features
    - [x] open_at: figure out how they're doing UNIX time

- [x] default to using just the business data
- [x] provide option to opt in to the web scraper
