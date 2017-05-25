---
layout: post
title:  CLI Data Scraper Project
date:   2017-05-24 20:02:04 -0400
---


I just finished my first real code-from-scratch project on Learn.co! The goal of the project was to use object-orientated Ruby to scrape and return data from a website using a command-line interface.

My app lets you input the year of an NHL season, and then it scrapes a website for data from that year, and returns the players with the most assists, points, and goals for that year. Then, if you input one of their names, it returns some more data about them, like their age and weight.

It sounds pretty simple, but it took me longer than I was expecting to complete the project. I had to research a lot of little things and I feel like I spent a lot of time just sitting there thinking about how everything comes together. Even just getting all my files and requirements set up properly took me a bit as I hadn't done that from scratch yet. 

Overall though, I feel like I learned a lot and It's pretty awesome how much more I can do now than I could just a month ago. I'm looking forward to getting my code reviewed to see what I can improve on!

#### Some Code

I used three classes. A CLI class, a Scraper class, and a Player class. 

When you run the program, a new instance of the CLI class is created which asks you for a year input. I grab this year and set it to an instance variable which I then use to create the URL for the page I want to scrape data from, using my Scraper.scrape_leaders class method.

```
@leaders << Scraper.scrape_leaders("http://www.hockey-reference.com/leagues/NHL_#{self.year}_leaders.html")
```

My Scraper class scrapes (using open-uri and nokogiri) the names and their associated number of assists, goals, and points for that year's leaders and adds the data to a hash.

```
def self.scrape_leaders(url)
    leader_doc = Nokogiri::HTML(open(url))

    leaders = {}
    leaders[:goal_leader_name] = leader_doc.css("div#leaders_goals .first_place .who a").first.text.strip
    leaders[:goal_leader_url] = BASE_PATH + leader_doc.css("div#leaders_goals .first_place .who a[href]").first["href"]
    leaders[:goals] = leader_doc.css("div#leaders_goals .first_place .value").first.text.strip
    leaders[:assist_leader_name] = leader_doc.css("div#leaders_assists .first_place .who a").first.text.strip
    leaders[:assist_leader_url] = BASE_PATH + leader_doc.css("div#leaders_assists .first_place .who a[href]").first["href"]
    leaders[:assists] = leader_doc.css("div#leaders_assists .first_place .value").first.text.strip
    leaders[:point_leader_name] = leader_doc.css("div#leaders_points .first_place .who a").first.text.strip
    leaders[:point_leader_url] = BASE_PATH + leader_doc.css("div#leaders_points .first_place .who a[href]").first["href"]
    leaders[:points] = leader_doc.css("div#leaders_points .first_place .value").first.text.strip

    leaders
end
```

It also scrapes a url that takes you to a page with more information about those players. I use this URL for each of the players to initialize new players using my Player class and then set their name, weight, height, etc. as attributes by scraping that URL.

```
def make_players
    Player.new("#{@leaders[0][:goal_leader_url]}")
    Player.new("#{@leaders[0][:assist_leader_url]}")
    Player.new("#{@leaders[0][:point_leader_url]}")
end
```

```
@@all = []

attr_accessor :name, :downcase_name, :height, :weight, :birthyear

def initialize(url)
  players_doc = Nokogiri::HTML(open(url))
  @downcase_name = players_doc.css(".players h1[itemprop='name']").text.gsub(" ", "").downcase
  @name = players_doc.css(".players h1[itemprop='name']").text
  @height = players_doc.css(".players span[itemprop='height']").text
  @weight = players_doc.css(".players span[itemprop='weight']").text
  @birthyear = players_doc.css(".players span[itemprop='birthDate'] a[2]").text
  @@all << self
end
```

I then return the goal leader, the assist leader, and the point leader using the CLI class and allow the user to input one of their names for more information about them, which I return using the Player's instance attributes (.height, .weight, etc).

```
def list_leaders
    puts "------------------------------"
    puts "#{self.year.to_s} NHL Season Leaders:"
    puts "------------------------------"
    puts "Goals: #{@leaders[0][:goal_leader_name]} - #{@leaders[0][:goals]}"
    puts "Assists: #{@leaders[0][:assist_leader_name]} - #{@leaders[0][:assists]}"
    puts "Points: #{@leaders[0][:point_leader_name]} - #{@leaders[0][:points]}"
    puts "------------------------------"

end
```
	
```
def player_details
	input = ""
	until input == "exit"
		puts "Enter one of above player's full name for more information about that player or exit"
		input = gets.chomp.gsub(" ", "").downcase
		if playerchoice = Player.all.detect {|player| player.downcase_name == input}
			puts "------------------------------"
			puts "#{playerchoice.name}"
			puts "------------------------------"
			puts "Height: #{playerchoice.height}"
			puts "Weight: #{playerchoice.weight}"
			puts "Born in: #{playerchoice.birthyear}"
			puts "------------------------------"
		elsif input == "exit"
			bye
		else
			puts "Invalid name"
		end
	end
end
```
