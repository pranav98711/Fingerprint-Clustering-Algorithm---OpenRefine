# Data Cleaning and Deduplication with Python - Fingerprint Clustering in OpenRefine

If you work with data - whether as a data scientist, analyst, engineer, or just in your daily tasks - you’ve probably realized that data cleaning and preprocessing are essential for accurate analysis. Last week, during my Information Visualization course, I learned a new way to clean messy data.  I got to know about OpenRefine (by Google), a tool that helps fix and organize data easily. One of its most interesting features is its clustering methods, especially the fingerprint algorithm. The tool is required to use the algorithm, but I wanted to make it more accessible. So, I converted the Java code to Python for easier use in my daily tasks.

### What is Fingerprint Clustering?
Imagine you have a dataset with many variations of the same name due to typos, extra spaces, or different formats. The fingerprint algorithm helps identify and group these similar values by converting them into a standardized key.

### How Does It Work?
The fingerprint algorithm transforms each value into a simplified version by following these steps:

1. Trim Extra Spaces – It removes any unnecessary spaces before or after the text.
2. Convert to Lowercase – Ensures that "John" and "john" are treated the same.
3. Remove Punctuation and Special Characters – Strips out symbols like commas, periods, and hyphens.
4. Normalize Special Characters – Converts accented characters to their simpler form (e.g., É → E or Gödel → Godel).
5. Split Words into Tokens – Breaks the text into individual words based on spaces.
6. Sort and Remove Duplicates – Organizes the words alphabetically and eliminates repeats.
7. Rejoin Words – Puts the cleaned-up words back together in a standard order.


### Let's see how we can implement this in real world scenario. 

Google Colab Code -> https://colab.research.google.com/drive/1ZEhe9pTaNsrnlEs9V2AVZJTkN8li6nhp#scrollTo=avXBW9bp4TOT

To see how fingerprint clustering works in practice, we can run it on a sample dataset using Google Colab. The dataset contains multiple variations of names, which can cause inconsistencies in data analysis.



<div align="center">
  <img src="https://github.com/user-attachments/assets/5da7e522-d77f-4c71-8511-c03bb3ed51e2" width="300">
</div>

When you run clustering algorithm on it 

```python
import unicodedata
import re
import pandas as pd
from collections import defaultdict

class FingerprintKeyer:
    PUNCTCTRL = re.compile(r"[^\w\s]", re.UNICODE)  # Removes all punctuation
    WHITESPACE = re.compile(r"\s+", re.UNICODE)  # Normalizes spaces
    
    NONDIACRITICS = {
        "ß": "ss", "æ": "ae", "ø": "oe", "å": "aa", "©": "c",
        "ð": "d", "đ": "d", "ɖ": "d", "þ": "th",
        "ƿ": "w", "ħ": "h", "ı": "i", "ĸ": "k",
        "ł": "l", "ŋ": "n", "ſ": "s", "ŧ": "t",
        "œ": "oe", "ẜ": "s", "ẝ": "s"
    }
    
    def key(self, s: str) -> str:
        if not isinstance(s, str):
            return None
        return " ".join(sorted(set(self.normalize(s, strong=True).split())))
    
    def normalize(self, s: str, strong: bool = False) -> str:
      """Normalizes text by removing diacritics, punctuation, and spaces"""
      if strong:
          s = s.strip().lower()
      
      # Ensure concatenated words (e.g., "JeanPaul") are properly spaced
      s = re.sub(r'(?<=[a-z])(?=[A-Z])', ' ', s)  # Adds space between lowercase-uppercase transitions

      # Replace problematic characters with spaces
      s = re.sub(r"[-.',_:/;!?()’]", " ", s)  # Handles all punctuation consistently

      s = self.strip_diacritics(s)
      s = self.strip_non_diacritics(s)
      s = self.PUNCTCTRL.sub("", s)  # Remove remaining punctuation
      s = self.WHITESPACE.sub(" ", s)  # Normalize multiple spaces to a single space

      tokens = s.split()
      tokens = sorted(set(tokens))  # Sort words and remove duplicates
      return " ".join(tokens)

    
    @staticmethod
    def strip_diacritics(text: str) -> str:
        """Removes diacritics (e.g., é → e, ñ → n)"""
        normalized = unicodedata.normalize('NFKD', text)
        return ''.join(c for c in normalized if not unicodedata.combining(c))
    
    def strip_non_diacritics(self, text: str) -> str:
        return ''.join(self.NONDIACRITICS.get(c, c) for c in text)

def cluster_data(file_path):
    """Reads an Excel file, applies fingerprinting, and clusters similar names"""
    
    # Load Excel file
    df = pd.read_excel(file_path)
    
    # Initialize Keyer
    keyer = FingerprintKeyer()
    
    # Dictionary to store clusters
    clusters = defaultdict(list)
    
    # Process each name
    for name in df["Names"]:
        key = keyer.key(name)
        clusters[key].append(name)
    
    # Convert clusters into a structured format
    cluster_list = [{"Cluster Key": key, "Names": ", ".join(variations)} for key, variations in clusters.items()]
    
    # Create a DataFrame for results
    clustered_df = pd.DataFrame(cluster_list)
    
    # Save clustered results to a new Excel file
    output_file = "clustered_names3.xlsx"
    clustered_df.to_excel(output_file, index=False)
    
    return output_file

# Example usage
if __name__ == "__main__":
    file_path = "test_names_variations.xlsx"  # Your input Excel file
    output_file = cluster_data(file_path)
    print(f"Clustered results saved at: {output_file}")
```


### Output

Even though the names are written differently, the fingerprint method converts them into the same key, helping to group them together. 


<div align="center">
  <img src="https://github.com/user-attachments/assets/73a91b58-952c-4d31-a7af-ed8c1e3c2ce9" width = "700">
</div>





The results aren’t perfect, but the fingerprint method still does a great job at grouping similar names. While some variations may not cluster exactly as expected, the overall process helps clean and standardize messy data effectively. With further refinements, it can be even more accurate and useful for real-world applications.

By applying fingerprint clustering, we can clean, standardize, and merge inconsistent text data efficiently. This not only improves data quality but also enhances searchability, analysis, and decision-making.


Read more about OpenRefine (Official Link) -> https://openrefine.org/docs/technical-reference/clustering-in-depth


