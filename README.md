import streamlit as st
import requests
import pandas as pd
import pydeck as pdk

st.set_page_config(layout="centered")
st.title("🌫️ India's Top 50 Cities - Real-time AQI")

# Your API key (reuse the same)
API_KEY = "c2c46476b711ec3bdd6ffc0f9be7b4c0"

# Top 50 cities with coordinates
cities_data = [
    ('Agra', 27.1767, 78.0081),
    ('Ahmedabad', 23.0225, 72.5714),
    ('Aligarh', 27.8974, 78.088),
    ('Allahabad', 25.4358, 81.8463),
    ('Amritsar', 31.634, 74.8723),
    ('Aurangabad', 19.8762, 75.3433),
    ('Bareilly', 28.367, 79.4304),
    ('Bengaluru', 12.9716, 77.5946),
    ('Bhopal', 23.2599, 77.4126),
    ('Chandigarh', 30.7333, 76.7794),
    ('Chennai', 13.0827, 80.2707),
    ('Coimbatore', 11.0168, 76.9558),
    ('Delhi', 28.6139, 77.209),
    ('Dhanbad', 23.7957, 86.4304),
    ('Faridabad', 28.4089, 77.3178),
    ('Guwahati', 26.1445, 91.7362),
    ('Gwalior', 26.2183, 78.1828),
    ('Howrah', 22.5958, 88.2636),
    ('Hubli-Dharwad', 15.3647, 75.124),
    ('Hyderabad', 17.385, 78.4867),
    ('Indore', 22.7196, 75.8577),
    ('Jaipur', 26.9124, 75.7873),
    ('Jabalpur', 23.1815, 79.9864),
    ('Jodhpur', 26.2389, 73.0243),
    ('Kanpur', 26.4499, 80.3319),
    ('Kalyan', 19.2403, 73.1305),
    ('Kolkata', 22.5726, 88.3639),
    ('Kota', 25.2138, 75.8648),
    ('Lucknow', 26.8467, 80.9462),
    ('Ludhiana', 30.9, 75.8573),
    ('Madurai', 9.9252, 78.1198),
    ('Meerut', 28.9845, 77.7064),
    ('Mumbai', 19.076, 72.8777),
    ('Mysuru', 12.2958, 76.6394),
    ('Nagpur', 21.1458, 79.0882),
    ('Nashik', 19.9975, 73.7898),
    ('Navi Mumbai', 19.033, 73.0297),
    ('Patna', 25.5941, 85.1376),
    ('Pune', 18.5204, 73.8567),
    ('Raipur', 21.2514, 81.6296),
    ('Rajkot', 22.3039, 70.8022),
    ('Ranchi', 23.3441, 85.3096),
    ('Solapur', 17.6599, 75.9064),
    ('Srinagar', 34.0837, 74.7973),
    ('Surat', 21.1702, 72.8311),
    ('Tiruchirappalli', 10.7905, 78.7047),
    ('Vadodara', 22.3072, 73.1812),
    ('Varanasi', 25.3176, 82.9739),
    ('Vasai-Virar', 19.3919, 72.8397),
    ('Vijayawada', 16.5062, 80.648)
]
city_df = pd.DataFrame(cities_data, columns=["City", "lat", "lon"])

# Get selected city via dropdown
selected_city = st.selectbox("Or choose a city manually", city_df["City"])

# Show map with points
st.pydeck_chart(pdk.Deck(
    initial_view_state=pdk.ViewState(latitude=22, longitude=78, zoom=4),
    layers=[
        pdk.Layer(
            "ScatterplotLayer",
            data=city_df,
            get_position='[lon, lat]',
            get_radius=8000,
            get_color='[255, 140, 0, 160]',
            pickable=True
        )
    ],
    tooltip={"text": "{City}"}
))

# Function to get AQI
def get_aqi(lat, lon, city_name):
    url = f"http://api.openweathermap.org/data/2.5/air_pollution?lat={lat}&lon={lon}&appid={API_KEY}"
    res = requests.get(url)
    if res.status_code != 200:
        st.error("Failed to fetch pollution data.")
        return
    data = res.json()
    aqi = data["list"][0]["main"]["aqi"]
    aqi_desc = {
        1: "Good 🌱",
        2: "Fair 🙂",
        3: "Moderate 😐",
        4: "Poor 😷",
        5: "Very Poor ☠️"
    }
    st.markdown(f"## 📍 Pollution in **{city_name}**")
    st.markdown(f"**AQI Index:** {aqi} ({aqi_desc.get(aqi, 'Unknown')})")

# Get city data from dropdown
city_data = city_df[city_df["City"] == selected_city].iloc[0]
get_aqi(city_data["lat"], city_data["lon"], city_data["City"])
