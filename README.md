# Officer Ollie

A small static website as documentation for our project for Major Studio 1 at MPS CD (Parsons School of Design). [https://officerollie.rijks.work](https://officerollie.rijks.work).

## Installation & Usage

Clone the repo and install the npm dependencies using `npm install`. 

### Production build the site

Run `npm run build`. This will use the markdown files in `src/site` to create the html pages of the website. It will also build the sass in `src/scss` and the static files in `src/public`.

### Deploy the site

Run `bash deploy.sh`

### Develop the site

By running `npm run dev`, you can work on the site locally. It will auto refresh on every change and re-build md changes automatically.

