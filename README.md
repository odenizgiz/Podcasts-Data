# Podcasts Data
Here you can find a dataset of approximately 10,000 podcasts that I collected from iTunes. 

The file, df_popular_podcasts.csv, is a dataframe which includes podcast name, the artwork, its genres, the number of the episodes, the duration of the episodes, three different associated URLs and the general description of the podcast. The  descriptions of all episodes of each podcast is in a text file, which is named after the particular podcast's position in the dataframe and can be found in the zipped files inside the /data/ folder. If you download and decompress 11 files here, you will get ~10,000 text files.


I also added 3 different Jupyter notebooks where you can see how exactly I collected this dataset. In the following, I will explain what is in each notebook, and the details of this dataset.   


## The Data
On the [iTunes website for podcasts](https://itunes.apple.com/us/genre/podcasts-arts/id1301?mt=2) there is the list of all genres and subgenres. For each genre and subgenre, the podcasts are grouped alphabetically from A to Z; but also, there is a list of "popular podcasts". The dataframe saved as df_popular_podcasts.csv includes the information of 10,155 of these popular podcasts. The columns of this dataframe are: 

* Name: The name of the podcast 

* Artwork: The link to the artwork of the podcast 

* Genre IDs: A list of genre IDs of the genres that a podcast is categorized in. I also provide what these IDs stand for in a separate file, called genre_IDs.txt.  

* Episode Count: The number of episodes released so far (August 2017) from a particular podcast. Of course, this number is changing, but should still be useful together with the next column "Episode Durations". Here, I limit the dataframe to contain only podcasts that have a minimum number of 20 episodes. 

* Episode Durations: A list of durations of each episode in minutes. 

* iTunes URL: The URL link to the podcast on iTunes. 

* Feed URL: The URL link of the RSS feed of the podcast. 

* Podcast URL: The URL link of the podcast's website. 

* Description: The general description of the podcast as written on its iTunes page. 

