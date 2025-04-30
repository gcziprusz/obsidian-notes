
Things to do 

- ~~Fork a student solution ~~
		- https://github.com/nashville-software-school/rock-of-ages-client-solution
		- https://github.com/nashville-software-school/rock-of-ages-api-solution
- Use pet adoption portal 
	- https://github.com/nashville-software-school/pet-adoption-portal
- run it on the host 
- look at how to Dockerize the API

	- pipenv inside a container
	- commands api build:
		`docker build -t pet-adoption-api .

	- command to run api in docker: 
		- `docker run -p 8000:8000 pet-adoption-api`
	- commands client 
		- `npm run build
	- migrations? - how to deal with that since API and DB should not be in same container
		- `python3 manage.py migrate` creates a sqllite3 db 
		
- look at how to dockerize the client 
- TBD??