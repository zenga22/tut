# Tut - Tablo User Tools <a name="tut"></a>

**Tut** lets you mess with your 
[Tablo](https://www.tablotv.com/). Some things it does:

* automatically finds your Tablo (there are options if you have multiples)
* builds a library of your recordings
* display stats about your library/recordings
* a large number of search options
* search/display _incomplete_ recordings

Matching searches can than be used to:
* **delete** recordings from your Tablo
* **copy** recordings to wherever   

Here's a fun example that _could_ be used to cleanup crappy recordings
on your Tablo:
```shell script
./tut.py -L library --incomplete 70 | ./tut.py delete
```
That will find a **show** with one or more recordings that only managed to record 70% of the show and delete them.   


##### Requirements
* Python 3 (tested against Python 3.10.6 on Ubuntu).

* Tested against Tablo firmware:
    * v 2.2.26
    * v 2.2.36   


### Installation
Download and unpack the 
[zip of this project](https://github.com/zenga22/tut/archive/master.zip) 

or clone it from [GitHub](https://github.com/zenga22/tut).
 
Go to the created directory and 
* run `pip3 install -r requirements.txt`

_You maybe want to run this in a [virtualenv](https://virtualenv.pypa.io/en/latest/)_


### Kick the wheels
With any luck after you've installed it, you can run these commands
and it'll do/display a bunch of stuff about your Tablo. See below for 
details on all sorts of other stuff you can do.
```
./tut.py config --discover
./tut.py library --build
./tut.py library --stats
./tut.py search --limit 2
```

### Detailed Usage
Run something like:

* `./tut.py`
* `python tut.py`
* `python3 tut.py`
 
and you should see something like:

```
usage: tut.py [-h] [--dry-run] [-L] [-v] [--version]
              {config,library,search,copy,delete} ...

Available commands:
  {config,library,search,copy,delete}
    config              manage configuration options
    library             manage the local library of recordings
    search              ways to search your library
    copy                copy recordings somewhere
    delete              delete recordings from the Tablo device

optional arguments:
  -h, --help            show this help message and exit
  --dry-run             show what would happen, but don't change anything
  -L, --id-list         if possible, only output a list of matching Object Ids
                        - Pipe this into other commands. (overrides --full and
                        any other output)
  -v, --verbose         amount of detail to display add vs (-vvvv) for maximum
                        amount
  --version             show program's version number and exit
```
#### Command Help

Running `./tut.py search --help` gives you:

```
usage: tut.py search [-h] [-t TERM] [-a AFTER] [-b BEFORE] [--limit LIMIT] [--season SEASON] [--episode EPISODE] [--state {finished,failed,recording}]
                     [--type {episode,movie,sport,programs}] [--duration DURATION] [--watched] [--protected] [--full] [--tms-id TMS_ID] [--id ID] [--casesensitive]

options:
  -h, --help            show this help message and exit
  -t TERM, --term TERM  search title/description for this
  -a AFTER, --after AFTER
                        only recordings after this date
  -b BEFORE, --before BEFORE
                        only recordings before this date
  --limit LIMIT         only recordings in this state
  --season SEASON       episodes with this season
  --episode EPISODE     episodes with this episode number
  --state {finished,failed,recording}
                        only recordings in this state
  --type {episode,movie,sport,programs}
                        only include these recording types
  --duration DURATION   recordings less than this length (28m, 10s, 1h. etc) - useful for culling bad recordings
  --watched             only include watched recordings
  --protected           only include protected recordings
  --full                dump/display full record details
  --tms-id TMS_ID       select by TMS Id (probably unique)
  --id ID               select by Tablo Object Id(definitely unique)
  --casesensitive       case-sensitive search for terms
```

### Configure
First things first, get your Tablo set up: 

`./tut.py config --discover`

Want to see some gory details of what happened there?

`./tut.py -vvv config --discover`  (more v, more info)

Then try:

`./tut.py config --view`

Possibly you want to look at the config file it told you exists?

### Your Library
Your "Library" is a local copy of recording data **at the time** you _build_ it.

##### Build it
Before you can do anything useful, you'll need to build the local cache/library of your recordings:

`./tut.py library --build`

A slow run including 630 recordings takes about 40 sec. 

##### General library stats 
`./tut.py library --stats`

And you'll see something like:
```
Overview
--------------------------------------------------
Total Recordings: 609
Total Watched: 65

By Current Recording State
--------------------------------------------------
Finished         : 577
Failed           : 32
Recording        : 0

By Recording Type
--------------------------------------------------
Episodes/Series  : 608
Movies           : 1
Sports/Events    : 0
Programs         : 0
```

##### Look for Incomplete recordings
This attempts to group recordings by the show/episode they belong to help determine what is unsalvageable. 
```shell script
./tut.py library --incomplete 
```   
*OR*, just find the obviously unsalvageable ones. (70 means 70% or the full show was recorded) 
```shell script
./tut.py library --incomplete 70 
```   
You can also pipe the output to delete them like so:
```shell script
./tut.py -L library --incomplete 70 | ./tut.py delete 
```

### Search (the Library)
There are a number of ways to search your library. This will be useful in specifying recordings you want to work with later.

**Important**: use the **-L** flag with _any_ search to create a list that can be 
piped into other opertaions.

Run `./tut.py search` to see the numerous options available.

A few examples follow. Note the combination of flags. 

###### All recordings with "colbert" (case insensitive) in the title or description:

`./tut.py search colbert`   

or:

`./tut.py search --term colbert`

###### Search for terms case-sensitive

`./tut.py search --term "NOVA"  --casesensitive`

###### Or limit that to only recordings after a specific date:

`./tut.py search colbert --after 2019-07-19`

###### Return all Failed recordings:

`./tut.py search --state failed`

###### Return at most 3 Failed recordings:

`./tut.py search --state failed --limit 3`

###### Return all the Movies:

`./tut.py search --type movie`

###### Return all the Movies, but dump the full data record:

`./tut.py search --type movie --full`

###### Find all recordings 30 seconds or shorter
`./tut.py -L search --duration 30s`

###### Find all recordings with protected status = True
`./tut.py -L search --protected`


### Copy/Archive recordings
Copy recordings off your Tablo. Currently there are no options to do anything but copy a full recording intact (compress more, downgrade, etc.).

Do a search, add the **-L** flag and pipe it into **copy**:

`./tut.py -L search colbert | ./tut.py copy `

Currently this doesn't:
    * try to put incomplete recordings together
        - as such, doesn't deal with TMS IDs to do that
        


### Delete recordings
Do a search, add the **-L** flag and pipe it into delete:

`./tut.py -L search --duration 30s | ./tut.py delete `

_Note_: you'll have to add a `--yes` flag to make the delete actually occur.


 
### Acknowledgements
 This wouldn't have been made without: 
 
 * [Original tut GitHub project by Jessedp](https://github.com/jessedp/tut)
 * [the code for the Kodi add-on](https://github.com/Nuvyyo/script.tablo) from the Nuvyyo folks. You'll find the slightly modified version of it  `tablo` module.
 * [TinyDB](https://github.com/msiemens/tinydb)
 
