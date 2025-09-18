# cord19_explorer.py

# -------------------------
# Imports
# -------------------------
import pandas as pd
import matplotlib.pyplot as plt
from wordcloud import WordCloud
import streamlit as st

# -------------------------
# Part 1: Data Loading
# -------------------------
@st.cache_data
def load_data(file_path):
    df = pd.read_csv(file_path)
    return df

df = load_data("metadata.csv")

# -------------------------
# Part 2: Data Cleaning & Preparation
# -------------------------
# Convert publish_time to datetime
df['publish_time'] = pd.to_datetime(df['publish_time'], errors='coerce')

# Extract year
df['year'] = df['publish_time'].dt.year

# Abstract word count
df['abstract_word_count'] = df['abstract'].fillna("").apply(lambda x: len(x.split()))

# Drop rows with missing title or publish_time
df_cleaned = df.dropna(subset=['title', 'publish_time'])

# -------------------------
# Part 3: Data Analysis
# -------------------------
# Publications by year
year_counts = df_cleaned['year'].value_counts().sort_index()

# Top 10 journals
top_journals = df_cleaned['journal'].value_counts().head(10)

# Word cloud from titles
all_titles = " ".join(df_cleaned['title'].dropna())
wordcloud = WordCloud(width=800, height=400, background_color='white').generate(all_titles)

# -------------------------
# Part 4: Streamlit App
# -------------------------
st.title("CORD-19 Data Explorer")
st.write("Interactive exploration of COVID-19 research papers")

# Show raw data sample
st.subheader("Sample of Data")
st.dataframe(df_cleaned.head())

# Interactive Year Slider
min_year = int(df_cleaned['year'].min())
max_year = int(df_cleaned['year'].max())
year_range = st.slider("Select year range", min_year, max_year, (2020, 2021))
filtered_df = df_cleaned[(df_cleaned['year'] >= year_range[0]) & (df_cleaned['year'] <= year_range[1])]

# Publications over time
st.subheader("Publications by Year")
filtered_year_counts = filtered_df['year'].value_counts().sort_index()
st.bar_chart(filtered_year_counts)

# Top journals bar chart
st.subheader("Top 10 Journals")
top_journals_filtered = filtered_df['journal'].value_counts().head(10)
st.bar_chart(top_journals_filtered)

# Word cloud of paper titles
st.subheader("Word Cloud of Paper Titles")
filtered_titles = " ".join(filtered_df['title'].dropna())
wordcloud_filtered = WordCloud(width=800, height=400, background_color='white').generate(filtered_titles)
st.image(wordcloud_filtered.to_array())

# Show basic statistics
st.subheader("Basic Statistics")
st.write({
    "Total Papers": len(filtered_df),
    "Average Abstract Word Count": filtered_df['abstract_word_count'].mean(),
    "Number of Journals": filtered_df['journal'].nunique()
})

# -------------------------
# Optional: Save cleaned data
# -------------------------
df_cleaned.to_csv("metadata_cleaned.csv", index=False)
