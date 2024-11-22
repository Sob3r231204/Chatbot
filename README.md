import re
import random
import requests
import nltk
from nltk.corpus import stopwords
from nltk.tokenize import word_tokenize
nltk.download('punkt_tab')
nltk.download('punkt')



# Download necessary NLTK data files (you can remove these after the first run if already downloaded)
nltk.download("punkt")
nltk.download("stopwords")

# Intents and their corresponding patterns/responses
intents = {
    "greeting": {
        "patterns": [r"hi", r"hello", r"hey", r"good morning", r"good evening"],
        "responses": ["Hello! How can I assist you today?", "Hi there! What can I do for you?", "Hey! How's it going?"]
    },
    "goodbye": {
        "patterns": [r"bye", r"goodbye", r"see you", r"exit"],
        "responses": ["Goodbye! Have a great day!", "See you soon!", "Bye! Take care!"]
    },
    "ask_weather": {
        "patterns": [r"weather in (.+)", r"what's the weather in (.+)", r"tell me the weather in (.+)"],
        "responses": ["Let me check the weather for you..."]
    },
    "ask_name": {
        "patterns": [r"your name", r"who are you", r"what are you called"],
        "responses": ["I'm your friendly chatbot!", "You can call me Chatbot.", "I'm here to assist you!"]
    },
    "default": {
        "patterns": [],
        "responses": ["I'm not sure I understand. Could you rephrase that?",
                      "I'm here to help, but I didn't quite get that."]
    }
}

# OpenWeatherMap API configuration
WEATHER_API_KEY = "8502e22f5be75669c6328cfad962bf63"

WEATHER_API_URL = "http://api.openweathermap.org/data/2.5/weather"

# Preprocessing user input
def preprocess_input(user_input):
    # Tokenize input
    tokens = word_tokenize(user_input.lower())
    # Remove stopwords
    stop_words = set(stopwords.words("english"))
    filtered_tokens = [word for word in tokens if word not in stop_words]
    # Reconstruct cleaned input
    return " ".join(filtered_tokens)

# Function to match user input to an intent
def match_intent(user_input):
    preprocessed_input = preprocess_input(user_input)
    for intent, intent_data in intents.items():
        for pattern in intent_data["patterns"]:
            match = re.search(pattern, preprocessed_input, re.IGNORECASE)
            if match:
                return intent, match
    return "default", None

# Function to fetch weather information
def get_weather(city):
    try:
        params = {"q": city, "appid": WEATHER_API_KEY, "units": "metric"}
        response = requests.get(WEATHER_API_URL, params=params)
        data = response.json()

        if response.status_code == 200:
            city_name = data["name"]
            temp = data["main"]["temp"]
            weather_desc = data["weather"][0]["description"]
            return f"The weather in {city_name} is currently {weather_desc} with a temperature of {temp}°C."
        else:
            return "I couldn't find the weather for that location. Please check the city name and try again."
    except Exception as e:
        return "Sorry, I encountered an error while fetching the weather. Please try again later."

# Function to generate a response
def generate_response(intent, match, context=None):
    if intent == "ask_weather" and match:
        city = match.group(1)
        return get_weather(city)
    if intent in intents:
        return random.choice(intents[intent]["responses"])
    return random.choice(intents["default"]["responses"])

# Main chatbot loop
def chatbot():
    print("ChatBot: Hi! I'm here to chat. Type 'exit' to end the conversation.")
    while True:
        user_input = input("You: ").strip()
        if user_input.lower() in ["exit", "quit"]:
            print("ChatBot: Goodbye! Have a great day!")
            break

        # Match the user input to an intent
        intent, match = match_intent(user_input)

        # Generate a response
        response = generate_response(intent, match)
        print(f"ChatBot: {response}")

# Run the chatbot
if __name__ == "__main__":
    chatbot()

