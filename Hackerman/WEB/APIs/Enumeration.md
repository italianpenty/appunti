### **Passive Reconnaissance**

*Google Dorks*

| ---                                                     | ---                                                                                                                                                |
| ------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Google Dorking Query**                                | **Expected results**                                                                                                                               |
| inurl:"/wp-json/wp/v2/users"                            | Finds all publicly available WordPress API user directories.                                                                                       |
| intitle:"index.of" intext:"api.txt"                     | Finds publicly available API key files.                                                                                                            |
| inurl:"/api/v1" intext:"index of /"                     | Finds potentially interesting API directories.                                                                                                     |
| ext:php inurl:"api.php?action="                         | Finds all sites with a XenAPI SQL injection vulnerability. (This query was posted in 2016; four years later, there are currently 141,000 results.) |
| intitle:"index of" api_key OR "api key" OR apiKey -pool | This is one of my favorite queries. It lists potentially exposed API keys.                                                                         |

### **# Active Reconnaissance**
