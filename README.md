# Podcasts Data
Here you can find a dataset of approximately 10,000 podcasts that I collected from iTunes. 

The file, df_popular_podcasts.csv, is a Pandas dataframe which includes podcast name, the artwork, its genres, the number of the episodes, the duration of the episodes, three different associated URLs and the general description of the podcast. The  descriptions of all episodes of each podcast is in a text file, which is named after the particular podcast's position in the dataframe and can be found in the zipped files inside the [/data](https://github.com/odenizgiz/Podcasts-Data/tree/master/data) folder. If you download and decompress 11 files here, you will get ~10,000 text files.


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


Apart from this dataframe, there are also additional data on these podcasts that you can find under the [/data](https://github.com/odenizgiz/Podcasts-Data/tree/master/data) folder. There are eleven compressed files in this folder, named such as 01_raw_data.zip, all of which include 10,155 text files, one for each podcast. Each text file contains all the titles and descriptions of all episodes as a bulk, including the general description, of a podcast, if they were available in its RSS feed (details on this later). For the cases when this information wasn't available, the corresponding text file is either left empty or only includes the word "empty". 

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

**Goal 2: extract information from iTunes API. Using the iTunes URLs that are collected in the previous step, I extracted the iTunes IDs of these podcasts.**  

<br><br/>


**Goal 3: to collect the second set of information from each podcast's iTunes page by parsing it.**  

I used again the iTunes URLs that are collected in the 1st step

 

<br><br/>

## 03. How the Data is Collected as the Text Files
