# Traffic Count

A simple tool for conducting intersection analysis by counting vehicles.

## Using

Once the app is deployed, open it in your browser. Set the name of the approach roads then use the displayed keyboard shortcuts to count vehicles. `A,W,D` count vehicles turning from the left approach and the left, up, and right arrows count vehicles from the right approach.  
Hold `Shift` and press a direction key to count a heavy vehicle or hold `Space` and press a direction key for a pedestrian or other vulnerable user.  
  
You may print the table or save as a PDF using either the normal keyboard shortcut (`Ctrl/Command+P`) or the "Print/Save as PDF" button at the top-right.  
You may download the raw data as a CSV (open in Excel) using the button at the top-right.  
You can reset all data by using the "Reset" button. You must then confirm that you want to (it's irreversable!) and it will completely reset the app.

## Developing

Once you've cloned the repository and installed dependencies with `npm install`, start a development server:

```bash
npm run dev

# or start the server and open the app in a new browser tab
npm run dev -- --open
```

## Building

To create a production version of the app:

```bash
npm run build
```

It will generate a statically-servable HTML/JS/CSS version of the site in the `build` directory.

You can preview the production build with `npm run preview`.
