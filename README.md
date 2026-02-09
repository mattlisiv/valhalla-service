# valhalla-service

## Instructions 
Clone Repo
```
git clone https://github.com/mattlisiv/valhalla-service.git .
```
Navigate to valhalla directory
```
cd valhalla
```

Create a custom files directory
```
mkdir custom_files && cd custom_files/
```

Download the US map data
```
wget https://download.geofabrik.de/north-america/us-latest.osm.pbf
```

Go back to root
```
cd ../..
```

Start docker services
```
docker-compose up -d
```

## Additional

It will take about 8 hours to initially populate the full US list 
