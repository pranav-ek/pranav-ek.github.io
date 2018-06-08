---
layout: post
title: Parse an XML document using SAX parser
date: '2013-11-27 08:24:19'
---

Simple API for XML(SAX) is an event driven parser which reads XML step by step. SAX Parser is faster and uses less memory than DOM parser.

Event handling methods used:
startElement() – Receive notification of the start of an element.
endElement() – Receive notification of the end of an element.
characters() – Receive notification of character data inside an element.


XML document to be parsed
<strong>movie_list.xml</strong>
{% highlight java %}
<movies>
    <movie>
        <name>The Terminal</name>
        <image>http://.../the_terminal.jpg</image>
        <description>terminal foo foo foo</description>
    </movie>
    <movie>
        <name>Shutter Island</name>
        <image>http://.../shutter_island.png</image>
        <description>shutter foo foo foo </description>
    </movie>
    <movie>
        <name>Old Boy</name>
        <image>http://.../old_boy.gif</image>
        <description>old foo foo foo </description>
    </movie>
</movies>
{% endhighlight %}


Create Model object
<strong>Movie.java</strong>
{% highlight java %}
public class Movie {

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public String getImgURL() {
		return imgURL;
	}

	public void setImgURL(String imgURL) {
		this.imgURL = imgURL;
	}

	public String getDescription() {
		return description;
	}

	public void setDescription(String description) {
		this.description = description;
	}

	private String name;
	private String imgURL;
	private String description;

	@Override
	public String toString() {
		return this.name + &quot; &quot; + this.imgURL + &quot; &quot; + this.description;
	}

}
{% endhighlight %}


Extend DefaultHandler and override above mentioned event handling methods.
<strong>MovieHandler.java</strong>
{% highlight java %}
public class MovieHandler extends DefaultHandler {
    List<Movie> movieList = new ArrayList<Movie>();
    Movie movie = null;
    String data = null;
 
    @Override
    public void startElement(String uri, String localName, String qName,
            Attributes attributes) throws SAXException {
               // When start tag of 'movie' element found, create an object and add it to the list
        switch (qName) {
        case "movie":
 
            movie = new Movie();
            movieList.add(movie);
            break;
 
        }
    }
 
    @Override
    public void endElement(String uri, String localName, String qName)
            throws SAXException {
 
        switch (qName) {
        case "name":
            movie.setName(data);
            break;
        case "description":
            movie.setDescription(data);
            break;
        case "image":
            movie.setImgURL(data);
            break;
 
        }
    }
 
    @Override
    public void characters(char[] ch, int start, int length)
            throws SAXException {
        data = String.copyValueOf(ch, start, length).trim();
 
    }
 
}
{% endhighlight %}


Set up the parser and get it started
<strong>MovieSaxParser.java</strong>
{% highlight java %}	
public class MovieSaxParser {
 
    static final Logger LOG = Logger.getLogger(MovieSaxParser.class);
    String static fileLocation = "/home/prnv/movie_list.xml";
 
    public static void main(String[] args) {
        try {
            SAXParserFactory saxFactory = SAXParserFactory.newInstance();
            SAXParser parser = saxFactory.newSAXParser();
            MovieHandler handler = new MovieHandler();
            parser.parse(fileLocation, handler);
            for (Movie movie : handler.movieList) {
                System.out.println(movie);
            }
        } catch (ParserConfigurationException | SAXException | IOException e) {
            LOG.error("Exception ", e);
 
        }
 
    }
}
{% endhighlight %}
