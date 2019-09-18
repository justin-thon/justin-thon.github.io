---
layout: post
title:  "PIN Code to lat long"
date:   2019-09-17 16:52:03 +0530
categories: jekyll update
---
You want to find the latitude and longitude of a particular pin code. One straightforward way is to google along with the keywords.
Say that of Connaught Place, New Delhi that has pin code 110001.

![Googoling pincode for lat long](/assets/cp.png)    

Hence, in theory you could scrape the html return page to get what we want i.e., lat/long.

But there's a more straight forward way. Just search the pin code in google maps.    

![Googling pincode in maps](/assets/cp1.png)


Hence, the return url includes what we want `(latitude and longitude)`.
In fact, if you enter `https://www.google.com/maps/place/110001` in your browser and hit return, it will lead to the same page above.

Thus, our api just have to hit the above url for any given pin code.

Below is the python code that does it:

{% highlight ruby %}
import requests

r=requests.get('https://www.google.com/maps/place/110001')

print(r.text)
{% endhighlight %}

The latitude and longitude are in the response text as seen below.

![Response text that contains lat/long](/assets/pystatic.png)

Close inspection of this output text reveals the pattern `"https://maps.google.com/maps/api/staticmap?center=28.6327426%2C77.2195969&amp;...`. We write a function that detects a substring and returns the index where it is found.

{% highlight ruby %}
# https://stackoverflow.com/questions/21842885/python-find-a-substring-in-a-string-and-returning-the-index-of-the-substring
# Eric Fortin's answer
def find_str(s, char):
    index = 0
    if char in s:
        c = char[0]
        for ch in s:
            if ch == c:
                if s[index:index+len(char)] == char:
                    return index
            index += 1
    return -1

print(find_str('Hellow World...', 'World'))
#=> 7 (index of W)
{% endhighlight %}

Noting that the latitude and longitude are separated by '%2C' and after a bit of trial and error (to extract exact index for these fields), we now can produce the full code below:

### Full code: pincode_to_location.py ###
{% highlight python %} 
import requests
import sys

# https://stackoverflow.com/questions/21842885/python-find-a-substring-in-a-string-and-returning-the-index-of-the-substring
# Eric Fortin's answer
def find_str(s, char):
    index = 0
    if char in s:
        c = char[0]
        for ch in s:
            if ch == c:
                if s[index:index+len(char)] == char:
                    return index
            index += 1
    return -1


def main():
	pin = sys.argv[1]
	r=requests.get('https://www.google.com/maps/place/'+pin,
		headers = {
		'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.86 Safari/537.36'
	    }
	    )
	page = r.text
	s="https://maps.google.com/maps/api/staticmap?center"
	s_i = find_str(page, s) + 50 
	s_e = s_i + 21
	loc = page[s_i:s_e].split('%2C')
	print(loc)
	return loc


if __name__=="__main__":
	main()

{% endhighlight %}  
Note: User Agent in requests is not necessary, we just don't want the google server to see (at least make it obvious) that a python code is calling them :)

Now we can run our program from the command prompt by typing:

#### 'python pincode_to_location.py 110001' that returns: ####
['28.6327426', '77.21959']