# ETL Python Project 
This is an ETL process involving Data from https://www.chess.com/games
## Reaquired Libraries
      import requests
      
      from bs4 import BeautifulSoup
      
      import pandas as pd
      
      import time
## Lets Start Data Extraction.
### use the appropriate url for web scraping
    player_names = []
    
    total_games = []
    
    player_url = []
    
    
    for i in range(1, 19):
    
        url = "https://www.chess.com/games?page=" + str(i)
        
        response = requests.get(url)
        
        soup = BeautifulSoup(response.text, "html.parser")
        
        names  = soup.find_all('a', class_= "post-preview-title")
        
        games = soup.find_all('div', class_= "post-preview-meta")
        
        for name in names:
        
            n = name.text.strip().replace("\n", "")
        
            player_names.append(n)
    
            p_url = "https://www.chess.com/games/" + str(n)
            
            player_url.append(p_url)
    
        for g in games:
            
            total_games.append(g.text.strip().replace("\n", ""))

### From the player_url list we get url for individual player. From this url, we can extract data related to that player

    player_age = []
    
    place_of_birth = []
    
    federation = []
    
    for link in player_url:
    
        response = requests.get(link)
        
        if response.status_code == 200:
        
            soup = BeautifulSoup(response.text, "html.parser")
            
            ages = soup.find_all("span", class_= "master-players-age")
            
            birth = soup.find_all("div", class_= "master-players-value")[2]
            
            fed = soup.find_all("div", class_= "master-players-value")[3]
            
            for age in ages:
            
                a = age.text.strip().replace("\u200e", "")
                
                player_age.append(a)
            
            for bi in birth:
            
                b = bi.text.strip().replace("\n", "")
                
                place_of_birth.append(b)
            
            for fe in fed:
            
                f = fe.text.strip().replace("\n", "")
                
                federation.append(f)
        
        else:
        
            print(response)
  
        print(a)
        
        time.sleep(15)

### The Reason why we added 'time.sleep(15)' is because, the response is getting '429', which means the server is overloaded

### Now check the lengths of these lists
    print(len(place_of_birth))
    
    print(len(federation))
    
    print(len(player_age))
    
    print(len(player_names))
    
    print(len(total_games))

## Transform
### Looks like 'player_age' and 'total_games' columns are not in proper format(DataType). Lets change that.
    final_player_age = [int(i.split()[1][:-1]) for i in player_age]
    
    final_total_games = [int(i.split()[0].replace(",", "")) for i in total_games]

## Load

### Create a pandas DataFrame and convert this DataFrame to required file type.
    df = pd.DataFrame({"Name": player_names, "Age": final_player_age, "Official Games Played": final_total_games, "Place of Birth": place_of_birth, "Federation": federation, })
    
    df.to_csv("ChessInfo.csv", index=False, header=True)
    
    df.to_excel("ChessInfo.xlsx", index=False, header=True)

