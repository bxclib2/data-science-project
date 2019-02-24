Since the Gmaps library is mostly designed for javascripts, the API interface of Python is limited. For using Gmaps for a desired visualization, we modified a little javascript source of the library.  
That means if you install gmaps with pip or conda, you won't acheive exactly the same results with us. To show the results given by us, you need to install gmaps with source code and change a little bit source code before compling.  
The whole process is not complicated, you can follow the instrutions as follows:  


1 First to make sure you have npm and node.js.  
You can simply use the package manager in your system. In Unbuntu, you can just type:
```
sudo apt-get install npm
sudo apt-get install node.js
sudo apt-get install nodejs-legacy
```

2 Clone the project from github.
```
git clone https://github.com/pbugnion/gmaps.git
```
3 Fix the bug from the install scripts.
From the source code, open the folder *js*, add a line
```
    "build":"webpack --config ./webpack.config.js",
```
in the file *package.json* (in the scripts block).

After adding, the scripts block should like this:
```json
  "scripts": {
    "build":"webpack --config ./webpack.config.js",
    "build:all": "npm run build:nbextension && npm run build:labextension",
    "build:watch": "webpack --watch",
    "build:nbextension": "webpack",
    "build:labextension": "cd ../gmaps && rimraf labextension && mkdirp labextension && cd labextension && npm pack ../../js",
    "clean:all": "npm run clean:nbextension && npm run clean:labextension",
    "clean:nbextension": "rimraf ../gmaps/static && rimraf dist/",
    "clean:labextension": "rimraf ../gmaps/labextension",
    "update": "rimraf dist/ && npm run build",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
```  

4 Change the source code.

From the source code, open the folder *js/src*, add 3 lines:
```
                    for (var i = 0; i < response.routes[0].legs[0].steps.length; i++) {
                    response.routes[0].legs[0].steps[i].transit = null
                    response.routes[0].legs[0].steps[i].instructions = null
                    }
```
in the file *Directions.js* (in the *computeDirections* function).


After adding, the function should like this:
```
    computeDirections() {
        const request = {
            origin: arrayToLatLng(this.model.get('start')),
            destination: arrayToLatLng(this.model.get('end')),
            waypoints: this.getWaypoints(),
            travelMode: this.model.get('travel_mode'),
            avoidFerries: this.model.get('avoid_ferries'),
            avoidHighways: this.model.get('avoid_highways'),
            avoidTolls: this.model.get('avoid_tolls'),
            optimizeWaypoints: this.model.get('optimize_waypoints')
        };

        return new Promise((resolve, reject) => {
            this.directionsService.route(request, (response, status) => {
                this.model.set('layer_status', status) ;
                this.touch()
                if (status == google.maps.DirectionsStatus.OK) {
                    for (var i = 0; i < response.routes[0].legs[0].steps.length; i++) {
                    response.routes[0].legs[0].steps[i].transit = null
                    response.routes[0].legs[0].steps[i].instructions = null
                    }
                    resolve(response)
                } else {
                    const errorMessage =
                        statusErrorMessages[status] ||
                        `Invalid status code: ${status}.`
                    this.mapView.model.events.trigger(
                        mapEventTypes.LAYER_ERROR,
                        MapEvents.layerError('directions', errorMessage)
                    )
                    reject(`Error returned by direction service: ${status}`)
                }
            })
        })
    }
```
5 Install.

In the gmaps main folder, run:
```
./dev-install
```

This installation guide is adapted from :

[http://jupyter-gmaps.readthedocs.io/en/latest/install.html#development-version](http://jupyter-gmaps.readthedocs.io/en/latest/install.html#development-version)