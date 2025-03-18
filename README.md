Fluent Bit Community Website
============================
This project generates the [Fluent Bit community website](https://fluentbit.io).


Getting started
---------------
To suggest changes to the website, follow the steps below.

**Prerequisites:** [Hugo v0.145.x](https://gohugo.io)

1. Fork this repository to your person git repository.

2. Run the following to build the Hugo development environment, watching the output for the provided 

```
    $ hugo server

    Watching for changes in [PATH_TO_PROJECT]/fluent-bit-website-hugo/{archetypes,assets,content,data,layouts,static}
    Watching for config changes in [PATH_TO_PROJECT]/fluent-bit-website-hugo/config.toml
    Start building sites … 
    
                         | EN   
      -------------------+------
        Pages            | 278  
        Paginator pages  |   0  
        Non-page files   |   0  
        Static files     | 312  
        Processed images |   0  
        Aliases          |   1  
        Cleaned          |   0  

     Built in 313 ms
     Environment: "development"
     Serving pages from disk
     Running in Fast Render Mode. For full rebuilds on change: hugo server --disableFastRender
     Web Server is available at //localhost:1313/ (bind address 127.0.0.1) 
     Press Ctrl+C to stop
```

3. Open the development version of the Fluent Bit website in your browser using the provided link above, in this case http://localhost:1313, and you should see the main Fluent Bit website landing page.

4. Make any changes you like to the code and test in the development server until finished, then save and commit to your forked repository.

5. Create a pull request from your forked repository to the main fluent bit website project and wait on feedback or acceptance of the changes.