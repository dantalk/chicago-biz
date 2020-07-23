# Exploring the Chicago business license data



<!--ts-->
   * [Exploring the Chicago business license data](chicago-intro.md#exploring-the-chicago-business-license-data)
      * [Intro](chicago-intro.md#intro)
      * [Tools and concepts](chicago-intro.md#tools-and-concepts)
      * [Real-world story/inspiration](chicago-intro.md#real-world-storyinspiration)
         * [Intro to the Chicago business license data](chicago-intro.md#intro-to-the-chicago-business-license-data)
      * [Making a hypothesis](chicago-intro.md#making-a-hypothesis)
      * [Getting started](chicago-intro.md#getting-started)
         * [Downloading the data](chicago-intro.md#downloading-the-data)
      * [Exploring the data](chicago-intro.md#exploring-the-data)
      * [A shallow test of data hypothesis with regex and line counting](chicago-intro.md#a-shallow-test-of-data-hypothesis-with-regex-and-line-counting)
         * [Bad assumptions and complicated data](chicago-intro.md#bad-assumptions-and-complicated-data)
      * [Proving our data hypothesis wrong with old-fashioned anecdotes](chicago-intro.md#proving-our-data-hypothesis-wrong-with-old-fashioned-anecdotes)

<!-- Added by: dan, at: Wed Jul 22 23:33:05 CDT 2020 -->

<!--te-->


## Intro 

This is a short walkthrough of how I use command-line tools to quickly and sanely work with data that's too large or cumbersome for Excel. For this lesson, I'll be using 

I'm practicing how to Twitch stream so unfortunately this isn't a full-fledged lesson – e.g. if you've never seen the command-line before, or plain text and comma-delimited files, you'll probably get lost quick.

> Warning: many of my commands/syntax will work as is in Windows PowerShell/Commandline, but many won't! Maybe at some point I'll repeat this in Windows but I don't have Twitch set up on my Windows side yet!



## Tools and concepts

Won't go into much depth here, but for easy reference:

- [regular expressions](https://www.regular-expressions.info/tutorial.html): a syntax for doing find-and-replace on steroids
- [ripgrep](https://github.com/BurntSushi/ripgrep), aka `rg`: a more powerful version of `grep` for searching text files with regex patterns
- [xsv](https://github.com/BurntSushi/xsv): a command-line tool for dealing with delimited text files – including regex searching and filtering by column (it's very similar to csvkit)
- [curl](https://curl.haxx.se/): a command-line tool for downloading from URLs

> Note: If you're on macOS, ripgrep, xsv, and curl are all easy-peasy to install with [homebrew](https://brew.sh/)

I also make brief use of these common *nix tools and commands: 

- `cd`
- `cat`
- `head`
- `sed`
- `sort`
- `tr`
- `uniq`
- `wc`


Also, I might occasionally show how to swiftly move data from command-line to Excel/Google Sheets, so you might see me use these macOS specific commands
- `open [something]` – Open `[something]` in the default application for opening things. For example, `open https://sheet.new` will pop open my Google Chrome to a new Google Sheet.
- `xlopen` – because Excel isn't the default app for `.csv` files, I use an aliased command: `alias xlopen="open -a 'Microsoft Excel'"` that lets me do: `xlopen data.csv`
- `pbcopy` - sometimes I want to get data into a Google Sheet as easy as possible without the normal point-and-click import process. I use `pbcopy` to read from a file/stream into my clipboard, and then I can just paste data right into Google Sheets: e.g. `xsv fmt '\t' data.csv | pbcopy`

## Real-world story/inspiration

From this Chicago Tribune story: [An estimated 4,400 Chicago-area businesses have closed during the pandemic. 2,400 say they’ll never reopen.](https://www.chicagotribune.com/coronavirus/ct-coronavirus-chicago-business-closures-yelp-20200722-nmhvpmv72fdyzdjgvzoun7rima-story.html)

> A tally is in: The coronavirus pandemic has forced an estimated 4,400 businesses in the Chicago area to close, including 2,400 that say they won’t reopen.

> The data, released Wednesday, comes from crowd-sourced business review platform Yelp.

> Nationally, more than 132,500 businesses have permanently or temporarily closed since March, according to Yelp. Temporary business closures are decreasing nationally as some states reopen, but permanent closures are rising, accounting for 55% of all closed businesses.

The story sources it's data to Yelp, but unfortunately Yelp's released data doesn't include Chicago. 

However, Chicago publishes a large database of business licenses, as far back as 2002:

https://data.cityofchicago.org/Community-Economic-Development/Business-Licenses/r5kz-chrr

Here's a sample of the data in Google Sheets for convenient viewing:
https://docs.google.com/spreadsheets/d/1U5_rDbDMZKLepB7IJKSrixzXN6Tuzdw-jt4jurML0o8/edit?usp=sharing


### Intro to the Chicago business license data

The documentation for the file can be found here:

https://data.cityofchicago.org/api/assets/D0EA58AC-19F2-4C5F-8668-ED6463F32B67?download=true

The relevant parts are: 

> APPLICATION TYPE: 

> - ‘ISSUE’ is the record associated with the initial license application. 
> - ‘RENEW’ is a subsequent renewal record. All renewal records are created with a term start date and term expiration date. 
> - ‘C_LOC’ is a change of location record. It means the business moved. ‘C_CAPA’ is a change of capacity record. Only a few license types may file this type of application. 
> - ‘C_EXPA’ only applies to businesses that have liquor licenses. It means the business location expanded. 

> LICENSE STATUS: 
> - ‘AAI’ means the license was issued. 
> - ‘AAC’ means the license was cancelled during its term.
> - ‘REV’ means the license was revoked. ‘REA’ means the license revocation has been appealed.

> LICENSE STATUS CHANGE DATE: This date corresponds to the date a license was cancelled (AAC), revoked (REV) or appealed (REA).


## Making a hypothesis

Just going by the brief documentation, and doing some googling, it might seem reasonable to assume that when a Chicago business closes, the owner [submits a cancellation notice to the city](https://www.chicago.gov/city/en/depts/bacp/sbc/update_your_businesslicense.html) to "cancel" their business license. 

And since the business license data has a `LICENSE STATUS` field that tracks cancellation – e.g. `AAC`, then the "algorithm" for calculating data could be as easy as:

- Filter license list to records in which the `LICENSE TERM EXPIRATION DATE` is *after* July/August 2020 (or whenever "today" is)
- Filter license list to records where `LICENSE STATUS` is `AAC`, since `AAC` ostensibly designates when a license was "canceled during its term"


> **Spoiler alert:** data doesn't care about your dumb/facile assumptions

## Getting started

For clarity during the stream, and to make it easier for you to copy-paste my commands at home – I'll be working in a self-contained subdirectory called: `mywork`:

```sh
$ mkdir mywork
$ cd mywork
```

### Downloading the data


The direct URL to download the business license data as CSV is:

https://data.cityofchicago.org/api/views/r5kz-chrr/rows.csv?accessType=DOWNLOAD

The file has a little over 1,000,000 rows and weighs in at 320MB+.


You can point-and-click to download and save like normal, but I like using `curl`:

```sh
$ curl -Lo raw_licenses.csv https://data.cityofchicago.org/api/views/r5kz-chrr/rows.csv?accessType=DOWNLOAD 
```



## Exploring the data


## A shallow test of data hypothesis with regex and line counting

According to the [documentation](assets/files/business_licenses_dataset_description.pdf), a `LICENSE STATUS` of `AAC` means "the license was cancelled during its term". So the following use of `xsv` will produce a subset of records where that is the case:

```sh
$ cat raw_licenses.csv \
    | xsv search 'AAC' -s 'LICENSE STATUS'
```

We *could* create a new CSV and explore it in a spreadsheet, but before we do that, why not do a quickie timeseries, by piping the output from above to another call to `xsv` – this time, to `select` just the `LICENSE TERM EXPIRATION DATE` column:



```sh
$ cat raw_licenses.csv \
    | xsv search 'AAC' -s 'LICENSE STATUS' \
    | xsv select 'LICENSE TERM EXPIRATION DATE' 
```

The output looks like this:
```
05/15/2016
04/15/2016
02/15/2007
02/15/2007
```

Doing a group count by day isn't helpful, so let's pipe the output into one more call of `rg` to produce a list of year-month values in `YYYY-MM` format:


```sh
$ cat raw_licenses.csv \
    | xsv search 'AAC' -s 'LICENSE STATUS' \
    | xsv select 'LICENSE TERM EXPIRATION DATE' \
    | rg  '(\d{2})/\d{2}/(\d{4})' -r '$2-$1'
```

Which gets us this:

```
2016-05
2016-04
2007-02
2007-02
```

Using good ol `sort` and `uniq`, we get a quickie group count:

```sh
$ cat raw_licenses.csv \
    | xsv search 'AAC' -s 'LICENSE STATUS' \
    | xsv select 'LICENSE TERM EXPIRATION DATE' \
    | rg  '(\d{2})/\d{2}/(\d{4})' -r '$2-$1' \
    | sort | uniq -c
```

Here's a relevant excerpt of the output, which should make it clear that counting by month may be too granular to tell how 2020 compares to 2019:

```
 164 2019-05
 208 2019-06
 280 2019-07
 188 2019-08
 209 2019-09
 174 2019-10
 162 2019-11
 165 2019-12
 102 2020-01
 158 2020-02
 103 2020-03
 124 2020-04
```

So we simply our previous call to just get the year:

```sh
$ cat raw_licenses.csv \
    | xsv search 'AAC' -s 'LICENSE STATUS' \
    | xsv select 'LICENSE TERM EXPIRATION DATE' \
    | rg  '(\d{2})/\d{2}/(\d{4})' -r '$2' \
    | sort | uniq -c
```

Without knowing too much about the data, something doesn't seem right or complete here, as the count for 2020 (even accounting for just half a year) doesn't seem wildly out of wack for what is a once-in-a-generation economic event:

```
2039 2016
1900 2017
2106 2018
2438 2019
1572 2020
 634 2021
  49 2022
```

### Bad assumptions and complicated data

But maybe we're close? The Chicago Tribune story/Yelp data said an "estimated 4,400 Chicago-area businesses have closed during the pandemic. 2,400 say they’ll never reopen". By our simple group count above, we see that ~2,250 licenses – set to expire in 2020 or later – have been canceled. 

But maybe we've made wrong assumptions?

- Lorem ipsum dolor sit amet, consectetur adipisicing elit. Adipisci fuga maiores consequuntur odit reprehenderit totam, sit rem, earum accusamus nesciunt ea cumque dicta enim quod ipsum? Fuga iure ut, quo!

## Proving our data hypothesis wrong with old-fashioned anecdotes

Let's take advantage of local reporting to find names of businesses that reporters have confirmed as going out of business; the [Chicago Eater has a running list of restaurants permanently closed because of the pandemic](https://chicago.eater.com/2020/5/12/21254955/chicago-restaurant-closings-coronavirus):


> River North: Nacional 27, Lettuce Entertain You Enterprises’s spot for Latin food, drinks, and dancing, permanently closed on March 1 after more than two decades, reps wrote on its website. The hospitality group opened up a Tallboy Taco spot inside the restaurant in 2014.



> River North: Katana, the pricey Japanese restaurant that arrived in Chicago three years ago, will not reopen its doors, a spokesperson confirmed to Eater Chicago in May. It was owned by Innovative Dining Group, a LA-based company that continues to offers delivery and takeout at its West Hollywood location. The restaurant’s opening came with much fanfare in 2017, as Hollywood celebrities frequented the West Coast location, and Chicago’s star chasers hoped to see that translate in the Midwest.


Here's how to use `xsv` to generate a quick CSV of business license records that contain either `Nacional 27` or `Katana`:


```sh
$ cat raw_licenses.csv | xsv search -i 'Nacional 27|katana' | xsv sort -s 'DOING BUSINESS AS NAME' > sample.closures.csv 
```


