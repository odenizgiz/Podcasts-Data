# Podcasts Data
Here you can find a dataset of approximately 10,000 podcasts that I collected from iTunes, plus a corpus of text which includes the full description of all episodes of these podcasts. 

The file, df_popular_podcasts.csv, is a Pandas dataframe which includes podcast name, the artwork, its genres, the number of the episodes, the duration of the episodes, three different associated URLs and the general description of the podcast. 

The corpus includes one text file for each podcast. The  descriptions of all episodes of each podcast is in this text file, which is named after the particular podcast's position in the dataframe and can be found in the zipped files inside the [/data](https://github.com/odenizgiz/Podcasts-Data/tree/master/data) folder. If you download and decompress 11 files here, you will get ~10,000 text files.


I also added 3 different Jupyter notebooks where you can see how exactly I collected this dataset. In the following, I will explain what is in each notebook, and the details of this dataset.   

<br><Br/>

## 01. General Information About the Data
On the [iTunes website for podcasts](https://itunes.apple.com/us/genre/podcasts-arts/id1301?mt=2) there is the list of all genres and subgenres. For each genre and subgenre, the podcasts are grouped alphabetically from A to Z; but also, there is a list of "popular podcasts". The dataframe saved as df_popular_podcasts.csv includes the information of 10,155 of these popular podcasts. The columns of this dataframe are: 

* Name: The name of the podcast. 

* Artwork: The link to the artwork of the podcast. 

* Genre IDs: A list of genre IDs of the genres that a podcast is categorized in. I also provide what these IDs stand for in a separate file, called genre_IDs.txt.  

* Episode Count: The number of episodes released so far (August 2017) from a particular podcast. Of course, this number is changing, but could still be useful. Here, I limit the dataframe to contain only podcasts that have a minimum number of 20 episodes. 

* Episode Durations: A list of durations of each episode of a podcast in minutes. 

* iTunes URL: The URL link to the podcast on iTunes. 

* Feed URL: The URL link of the RSS feed of the podcast. 

* Podcast URL: The URL link of the podcast's website. 

* Description: The general description of the podcast as written on its iTunes page. 


<p><p/> 


Apart from this dataframe, there is also a corpus of text that you can find under the [/data](https://github.com/odenizgiz/Podcasts-Data/tree/master/data) folder. There are eleven compressed files in this folder, named such as 01_raw_data.zip, all of which include 10,155 text files, one for each podcast. Each text file contains all the titles and descriptions of all episodes as a bulk, including the general description, of a podcast, if they were available in its RSS feed (details on this later). For the cases when this information wasn't available, the corresponding text file is either left empty or only includes the word "empty". 

The name of the text file corresponds to the location of the podcast in the dataframe, i.e. if a podcast is in the first row in the dataframe, which is indexed as 0; then its text file is named as 0.txt. 


<br><Br/>


## 02. How the Data is Collected for the Dataframe 
I extracted the data for the columns, 'Name', 'Episode Count', 'Episode Durations', 'iTunes URL', 'Podcast URL', and 'Description', by parsing the iTunes page of the particular podcast. While for the other columns, 'Artwork', 'Genre IDs' and 'Feed URL', I obtained the data by querying the iTunes API. The main code for this can be found under the file: [02.1 Build the Dataset from iTunes Website.ipynb](https://github.com/odenizgiz/Podcasts-Data/blob/master/02.1%20Build%20the%20Dataset%20from%20iTunes%20Website.ipynb).

Here I describe step by step how I proceeded:

**Goal 1: extract the iTunes URLs of all the podcasts under 'Popular Podcasts'.**  

* Step 1: First thing I did towards this was to extract all the links [here](https://itunes.apple.com/us/genre/podcasts/id26?mt=2) on the iTunes website using BeautifulSoup.

        url = web.URL(webpage)
        bs = BeautifulSoup(url.download(cached = False)) 
        links_page = []
        for link in bs.findAll('a', href=True):
            links_page.append(link['href'])

* Step 2: But what we want to do is to get all the links to the particular genres and subgenres because that is where the "Popular Podcasts" are listed. These genre links follow a particular pattern: for example, the link to the genre 'Arts'
is:

    https://itunes.apple.com/us/genre/podcasts-arts/id1301?mt=2 

   whereas the link to the genre 'Business' is: 
 
    https://itunes.apple.com/us/genre/podcasts-business/id1321?mt=2

   Therefore, I used regular expressions to extract from all the links the ones that match the pattern:    'https://itunes.apple.com/us/genre/podcasts-', which you can easily do with two lines of code:  


        if re.search(pattern, link):
             pattern_links.append(link)


* Step 3: In order to collect individual podcast links, I repeated steps 1 & 2 now with all these links at hand, but this time with the pattern "https://itunes.apple.com/us/podcast/". The reason being is that all podcast links start with this pattern followed by the name of the podcast and its iTunes ID. i.e. the iTunes link of the podcast "The Tim Ferriss Show" is: https://itunes.apple.com/us/podcast/the-tim-ferriss-show/id863897795?mt=2 

     After removing the links that may have repeated, I saved the remaining links using Pickle, which is named as       'popular_podcasts_links' that you can find in this directory. 
  
 

<br><br/>

**Goal 2: to collect information from each podcast's iTunes page by parsing it.**  

The iTunes URLs that are collected in the previous step can be used to collect crucial information about each podcast by crawling into each web page. After a little inspection, the code required to do this is pretty straightforward. For example, the name of a podcast can be extracted as: 
          
            
            url = web.URL(webpage)
            bs = BeautifulSoup(url.download(cached = False)) 
            titles = bs.find('div', id='title')
            if titles is not None:
                title = titles.find('h1').getText()

The link to the podcast's website (on the left column), its description and episode durations can all be obtained in a similar fashion. You can find the exact code in the file, [02.1 Build the Dataset from iTunes Website.ipynb](https://github.com/odenizgiz/Podcasts-Data/blob/master/02.1%20Build%20the%20Dataset%20from%20iTunes%20Website.ipynb). 

This step is especially necessary since neither the description of the podcasts, nor the podcast URL link are not provided through the iTunes API. However, in order to collect more information, such as all the podcasts episodes notes in full, the RSS feed of the podcasts should be parsed (on the iTunes website there is only summaries). And the links to the RSS feeds are provided by the iTunes API.      

<br><br/>


**Goal 3: extract information from iTunes API.** 

* Step 1: How to access iTunes API is explained [here](https://affiliate.itunes.apple.com/resources/documentation/itunes-store-web-service-search-api/). I decided to look up for the podcasts by their iTunes ID instead of searching for the podcast name since the former is unique. The iTunes URLs that are collected in the first part contain IDs associated with each podcast, i.e. for 'The Tim Ferriss Show', https://itunes.apple.com/us/podcast/the-tim-ferriss-show/id863897795?mt=2, the ID is _863897795_. 

     Using again regular expressions, we can extract the IDs: 

           podcast_id = re.findall('[0-9]+', re.findall('id[0-9]+', iTunes_URL)[0])[0]



* Step 2: Now these IDs can be used to query the iTunes API for a particular podcast. Leaving ID as the parameter, we can look up a particular podcast as: 

           url = 'https://itunes.apple.com/lookup?id=' + ID
           response = requests.get(url) 
           data = response.json()

    The result returned is in JSON format, and includes a dictionary with keys 'resultCount' and 'results'. The value for
    'results' key is a list of dictionaries for each search result.

        {'resultCount': 1,
         'results': [{'artistId': 867667252,
           'artistName': 'Tim Ferriss: Bestselling Author, Human Guinea Pig',
           'artistViewUrl': 'https://itunes.apple.com/us/artist/tim-ferriss/id867667252?mt=2&uo=4',
           'artworkUrl100': 'http://is5.mzstatic.com/image/thumb/Music127/v4/00/aa/e9/00aae9d6-1484-0d65-c70b-11132773bcae/source/100x100bb.jpg',
           'artworkUrl30': 'http://is5.mzstatic.com/image/thumb/Music127/v4/00/aa/e9/00aae9d6-1484-0d65-c70b-11132773bcae/source/30x30bb.jpg',
           'artworkUrl60': 'http://is5.mzstatic.com/image/thumb/Music127/v4/00/aa/e9/00aae9d6-1484-0d65-c70b-11132773bcae/source/60x60bb.jpg',
           'artworkUrl600': 'http://is5.mzstatic.com/image/thumb/Music127/v4/00/aa/e9/00aae9d6-1484-0d65-c70b-11132773bcae/source/600x600bb.jpg',
           'collectionCensoredName': 'The Tim Ferriss Show',
           'collectionExplicitness': 'cleaned',
           'collectionHdPrice': 0,
           'collectionId': 863897795,
           'collectionName': 'The Tim Ferriss Show',
           'collectionPrice': 0.0,
           'collectionViewUrl': 'https://itunes.apple.com/us/podcast/the-tim-ferriss-show/id863897795?mt=2&uo=4',
           'contentAdvisoryRating': 'Clean',
           'country': 'USA',
           'currency': 'USD',
           'feedUrl': 'http://timferriss.libsyn.com/rss',
           'genreIds': ['1412', '26', '1321', '1304', '1307'],
           'genres': ['Investing', 'Podcasts', 'Business', 'Education', 'Health'],
           'kind': 'podcast',
           'primaryGenreName': 'Investing',
           'releaseDate': '2017-08-24T12:07:00Z',
           'trackCensoredName': 'The Tim Ferriss Show',
           'trackCount': 262,
           'trackExplicitness': 'cleaned',
           'trackHdPrice': 0,
           'trackHdRentalPrice': 0,
           'trackId': 863897795,
           'trackName': 'The Tim Ferriss Show',
           'trackPrice': 0.0,
           'trackRentalPrice': 0,
           'trackViewUrl': 'https://itunes.apple.com/us/podcast/the-tim-ferriss-show/id863897795?mt=2&uo=4',
           'wrapperType': 'track'}]}


    From this result we see that the keys necessary to collect RSS feed URL, and genre IDs are 'feedUrl' and 'genreIds'. 
    For the artwork, there are three keys corresponding to three different sizes. I chose 'artworkUrl100'. 
    
    
    For more details on the code, see
    [02.1 Build the Dataset from iTunes Website.ipynb](https://github.com/odenizgiz/Podcasts-Data/blob/master/02.1%20Build%20the%20Dataset%20from%20iTunes%20Website.ipynb)

<br><br/>



 



## 03. How the Data is Collected as the Text Files

The descriptions of each episode of a podcast cannot fully be scraped from the iTunes website, but they are available - most of the time - on the RSS feeds of the podcasts. After collecting all the feed URLs from the iTunes API, I put them in the dataframe, as explained in the previous section. The code where each feed is crawled can be found in the notebook [02.2 Extract Raw Data.ipynb](https://github.com/odenizgiz/Podcasts-Data/blob/master/02.2%20Extract%20Raw%20Data.ipynb).

The important thing to note is that RSS feeds are in xml format, so you have to specify while scraping via BeautifulSoup:

        url = web.URL(webpage)
        bs = BeautifulSoup(url.download(cached = False), features='xml') 

However, some of the text will still have html tags inside after I extracted them via ```getText()```, therefore I passed these again through the function ```BeautifulSoup()```: 
                
        titles = bs.findAll('title')
        descriptions = bs.findAll('description')

        if (titles is not None) and (titles != []): 
            for title in titles:
                title = title.getText()
                if "<" in title:  
                    bs2 = BeautifulSoup(title)
                    title = bs2.getText() 
                    if title not in podcast_titles:
                        with open(str(i) + '.txt','a') as t:
                            t.write('\n' + title)      
                            t.close()
                    ... 
                    

<br><br/>

## 04. Building the Dataframe through iTunes API directly

At the beginning when I decided to collect the podcast data, I turned immediately to the iTunes API. I realized that there wasn't a direct way to collect a large dataset from the API, since you can only make a search with specific terms or look up podcasts if you already know enough information about them ([see here](https://affiliate.itunes.apple.com/resources/documentation/itunes-store-web-service-search-api/)). Not realizing that there was an explicit list of podcasts already on the iTunes website (eventually this is how I collected the names/urls of the "popular podcasts" as explained in the previous sections), I thought I could collect the data from API using the following strategies: 

**Strategy 1: Search for content using genres as search terms:** 
  
  * Step 1: 
    You can make a query to the API by searching for specific terms. These terms can be anything, but in order to get a
    homogeneous result, I put several possible genre names as the search term: 

                def build_genres_data(self, genre):

                        url = "https://itunes.apple.com/search?term=" + genre + "&limit=200&country=US&lan=en_us&entity=podcast"
                        response = requests.get(url) 
                        data = response.json()
                        ... 


    In the ```url``` above, you can see that I limited my search to US, picked English as the language; and more importantly 
    set the ```entity``` as podcast so that I only get podcasts in the result. iTunes API has the limit of maximum 200 on the 
    search results, and you have to specify it, otherwise it only gives a small number. 

  
  * Step 2:
    Again assuming that we do not know about the iTunes website which lists all the genres, how do we know which genres exist       
    from the API? It is important here to note that every podcast result comes with a key "genres", which is a list of genres
    that this podcast is categorized in (See Section 2, Goal 3), for example: 

                {'resultCount': 1,
                 'results': [{'artistId': 867667252,
                              'artistName': 'Tim Ferriss: Bestselling Author, Human Guinea Pig',
                              'genres': ['Investing', 'Podcasts', 'Business', 'Education', 'Health']
                 ...

    That is why, I first initialized my code with the search term "podcasts". Consider the function in Step 1 only with a 
    different ```url```:

        url = "https://itunes.apple.com/search?term=podcasts&limit=200&country=US&lan=en_us&entity=podcast"

    The API returned 200 podcast results which I used to feed another function that would extract the genres associated with   
    all of them. This list of genres would be the first seed to use in Step 1 to collect podcasts. But what if there are more
    genres? As I collected more podcasts, the code can check each of their genres, and if it finds one that is not already in         
    the list of genres, it would add it to it; and use it later to search for more podcasts. In total I found 68        
    genres/subgenres, which is just 7 more than the number of genres I got from the initialization. 


   * Result:
    By using this procedure, I collected about ~5,000 podcasts. It is important to note that for a particular search term, the
    API returned exactly the same result of 200 podcasts (even though there might be more podcasts associated with the term).
    Otherwise. one could have also sampled 200 podcasts many times for one term. You can find the code in full in the notebook    
    [01. Build the Dataset from Genres.ipynb](https://github.com/odenizgiz/Podcasts-Data/blob/master/01.%20Build%20the%20Dataset%20from%20Genres.ipynb)
  
             



**Strategy 2: Search for content using random search terms:**

Another idea was to use random set of 1000 words from one of the corpora available in the [NLTK website](http://www.nltk.org/book/ch02.html) for search. After trying a sample of 30 words, the number of results turned back from the API varied a lot from word to word. Most of the time, it was actually zero. That is why I decided not to use random words to generate podcasts. However, a function still exists for building data based on this in the notebook [01. Build the Dataset from Genres.ipynb](https://github.com/odenizgiz/Podcasts-Data/blob/master/01.%20Build%20the%20Dataset%20from%20Genres.ipynb).  






