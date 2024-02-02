from bs4 import BeautifulSoup
import requests

from selenium import webdriver
from selenium.webdriver.common.by import By
from webdriver_manager.chrome import ChromeDriverManager

while True:
    city_choice = input('\nWhich city are you interested in? \n').strip()
    print()
    state_choice = input('What state is that city in? (full name please): ').strip()
    
    print()
    print('Here\'s some info on ' + city_choice.capitalize() + ', ' + state_choice.capitalize() + '...') 

    # If both the city and state are each one word, then NO CONDITIONALS are needed!
    url = 'https://www.city-data.com/city/' + city_choice.capitalize() + '-' + state_choice.capitalize() + '.html'

    # These 2 variables are created for the conditionals that follow
    city_choice_dashes = ''
    state_choice_dashes = ''

    # Conditional for when the city and state are BOTH multiple words long each
    if len(city_choice.split()) > 1 and len(state_choice.split()) > 1:
        for word in city_choice.split():
            word = word.capitalize()
            city_choice_dashes += word + '-' 
        for word in state_choice.split():
            word = word.capitalize()
            state_choice_dashes += word + '-'
        url = 'https://www.city-data.com/city/' + city_choice_dashes[:-1] + '-' + state_choice_dashes[:-1] + '.html'
    
    # Conditional for when only the CITY is multiple words long (won't be reached if the above conditional is met)
    elif len(city_choice.split()) > 1:
        for word in city_choice.split():
            word = word.capitalize()
            city_choice_dashes += word + '-' 
        url = 'https://www.city-data.com/city/' + city_choice_dashes[:-1] + '-' + state_choice.capitalize() + '.html'
    
    # Conditional for when only the STATE is multiple words long (won't be reached if the above conditional is met)
    elif len(state_choice.split()) > 1:
        for word in state_choice.split():
            word = word.capitalize()
            state_choice_dashes += word + '-'
        url = 'https://www.city-data.com/city/' + city_choice.capitalize() + '-' + state_choice_dashes[:-1] + '.html'

    # If the city and/or state is not found, then there will be an error, which will trigger the "except" statement!
    try:
        page = requests.get(url)
        soup = BeautifulSoup(page.text, 'lxml')
        
        def find_weather():
            driver = webdriver.Chrome(ChromeDriverManager().install())
            driver.get(url)
            driver.maximize_window()
            
            weather_html = driver.find_elements(By.CLASS_NAME, 'wxbox')
            weather_data = ''
            for weather in weather_html:
                weather_data += weather.text
            
            weather_list = weather_data.split('\n')
            weather_list[0] = 'Temperature: ' + weather_list[0] # Adding "temperature" to the data
            weather_list[1] = 'Visibility: ' + weather_list[1] # Adding "visibility" to the data
            
            for weather in weather_list:
                print(weather) # Prints out each part of the weather data in the city
            
        def find_median_income():
            """Returns the median city and state income"""
            income = soup.find('section', class_='median-income')
            income_list = income.find_all('td')
            
            median_city_income = income_list[1].text
            median_state_income = income_list[3].text
            
            return city_choice.capitalize().strip() + ' median income: ' + median_city_income + '\n' +\
                state_choice.capitalize().strip() + ' median income: ' + median_state_income
            
        def find_median_house_value():
            """Returns the median house value in the city and state"""
            house = soup.find('section', class_='median-income')
            house_list = house.find_all('td')
            
            median_city_house = house_list[5].text
            median_state_house = house_list[7].text
    
            return city_choice.capitalize().strip() + ' median house value: ' + median_city_house + '\n' +\
                state_choice.capitalize().strip() + ' median house value: ' + median_state_house
    
        def find_racial_demographics():
            """Prints race and percentage of that race in the city inputted"""
            race_percentages_list = []
            race_names_list = []
            
            races = soup.find('ul', class_='list-group')
            race_percentages = races.find_all('span', class_='badge alert-info')
            race_names = races.find_all('b')
            
            # Contains list of all race percentages
            for percent in race_percentages:
                race_percentages_list.append(percent.text) 
                
            # Contains list of all race names
            for race in race_names:
                race_names_list.append(race.text)
                
            # Changes one of the race names because it shows up as "Native Hawiian and OtherPacific Islander alone"    
            for idx, race_name in enumerate(race_names_list):
                if 'Native Hawaiian and OtherPacific Islander alone' in race_name:
                    race_names_list[idx] = 'Native Hawaiian and Other Pacific Islander alone' 

            # Adds each race name and percentage into a dictionary
            race_dict = {}
            for key in race_names_list:
                for value in race_percentages_list:
                    race_dict[key] = value
                    race_percentages_list.remove(value)
                    break
            
            for k, v in race_dict.items():
                print(k + ': ' + v)
                
        # I have to declare the functions here in order for the "except" statement to run!
        print('\n---WEATHER---')
        find_weather() 
        
        print('\n---MEDIAN INCOME---')
        print(find_median_income())
        
        print('\n---MEDIAN HOUSE VALUE---')
        print(find_median_house_value())
        
        print('\n---RACIAL DEMOGRAPHICS---')
        find_racial_demographics()
        
    except:
        print()
        print('City and/or state was not found in the database. PLEASE TRY AGAIN.\n')
        continue
    
    try_again = input('\nDo you want to search up another city (Type Y for yes or N for no): ').upper().strip()
    
    # If the user types something other than yes or no, then the program will keep telling them to enter a valid response.
    while not (try_again == 'N' or try_again == 'NO') and not (try_again == 'Y' or try_again == 'YES'):
        print('\nThat\'s not a valid response. Try again:')
        try_again = input('\nDo you want to search up another city (Type Y for yes or N for no): ').upper().strip()
    if try_again == 'Y' or try_again == 'YES':
        continue
        print()
    else:
        print('\nThank you for using the program!')
        break