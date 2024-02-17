## Tagging favourite movies.

On the Home page, we can tag a movie as one of our favourites by clicking its 'Favourite' icon. This tagging will cause the related card to show a red Favorite Icon on the title bar. As a temporary solution, we will record a tagging by adding a 'favourite' property to the movie object and set it to true. [A more realistically approach is to persist them to a database via an API.] 

![][favorites]

We will again use the inverse data flow (Data down, Action up) pattern, this time to handle when a user tags a movie as a favourite. We record this fact by adding a 'favourite' property to the movie object and set it to true. In `pages/homePage.tsx`, make the following changes to the component:

```
..... code as before ......
​
const MovieListPage = (props) => {
  ..... code as before .......
​
  // New function
  const addToFavourites = (movieId) => {
    const updatedMovies = movies.map((m) =>
      m.id === movieId ? { ...m, favourite: true } : m
    );
    setMovies(updatedMovies);
  };
​
  useEffect(() => {
    ..... code as before .....
  }
  return (
    ...... code as before .....
```

Also, replace the line:

```
   <MovieList movies={displayedMovies} />
```

with this:

```
    <MovieList movies={displayedMovies} selectFavourite={addToFavourites} />
```

In `components/movieList/indes.jsx`, replace the line:

```
    <Movie key={m.id} movie={m} />
```

with this:

```
    <Movie key={m.id} movie={m} selectFavourite={props.selectFavourite} />
```

In `components/movieCard/index.jsx`, make the following changes:

- add an event handler:

```
export default function MovieCard(props) {
  ..... code as before .......
​
  const handleAddToFavourite = (e) => {
    e.preventDefault();
    props.selectFavourite(movie.id);
  };
​
  return (
    ...... code as before .....
```

- replace the lines:

```
    <IconButton aria-label="add to favourites" onClick={null}>
      <FavoriteIcon color="primary" fontSize="large" />
    </IconButton>
```

with this:

```
    <IconButton aria-label="add to favourites" onClick={handleAddToFavourite}>
        <FavoriteIcon color="primary" fontSize="large" />
    </IconButton>
```

- add an import at the top:

```
import Avatar from "@mui/material/Avatar";
```

- and replace the line:

```
    <CardHeader sx={styles.header} title={movie.title} />
```

with this:

```
    <CardHeader
      sx={styles.header}
      avatar={
        movie.favourite ? (
          <Avatar sx={styles.avatar}>
            <FavoriteIcon />
          </Avatar>
        ) : null
      }
      title={
        <Typography variant="h5" component="p">
          {movie.title}{" "}
        </Typography>
      }
    />
```

Until now, a component prop is either a data object or a callback functions. However, in the last excerpt above, a component is the value assigned to the title and avatar props. This is acceptable because a component transpiles to a function. Later we will develop custom components that accept other components as props.

The above changes cause the following runtime execution flow:

1. On the Home page, the user clicks a movie card's Favourite icon.
2. The event handler invokes the callback provided by the HomePage. 
3. The callback sets the movie's favourite property to true - a state change. 
4. This state change causes the HomePage to re-render. 
5. The tagged MovieCard displays a (red) Favourite icon in its title.   

Try selecting some movies as your favourites.

We need a new view to display the list of favourites. Its layout will be identical to the home page. Implementing this without violating the DRY principle (Don't Repeat Yourself) will require some refactoring - see next section. As a placeholder, create the file `src/pages/favouriteMoviesPage.jsx` and add this code:

## 
import React from "react";

const FavouriteMoviesPage = () => {
    return <h2>Favourite Movies</h2>
}

export default FavouriteMoviesPage
##

We must update the routing configuration and add a simple site navigation bar. In `src/index.jsx`, completely replace the content with the following:

```
import React from "react";
import { createRoot } from "react-dom/client";
import { BrowserRouter, Route, Navigate, Routes, Link } from "react-router-dom";
import HomePage from "./pages/homePage";
import MoviePage from "./pages/movieDetailsPage";
import FavouriteMoviesPage from "./pages/favouriteMoviesPage"; // NEW

const App = () => {
  return (
    <BrowserRouter>
      <ul>
        <li>
          <Link to="/">Home</Link>
        </li>
        <li>
          <Link to="/movies/favourites">Favourites</Link>
        </li>
      </ul>
      <Routes>
        <Route path="/movies/favourites" element={<FavouriteMoviesPage />} />
        <Route path="/movies/:id" element={<MoviePage />} />
        <Route path="/" element={<HomePage />} />
        <Route path="*" element={<Navigate to="/" />} />
      </Routes>
    </BrowserRouter>
  );
};

const rootElement = createRoot(document.getElementById("root"));
rootElement.render(<App />);
```

We must restart the development server whenever we nake routing configuration changes:

```
$ npm run dev
```

In the browser, check the new navigation links at the top are working.

The app has a bug! Try the following sequence:

1. Go to the home page.

2. Click the 'Favourites' icon of the first three movies.
3. Notice, the selected movies are tagged as expected. 
4. Go to the Favourites page, using the navigation links on top
5. Navigate back to the Home page. The three selected movies are no longer tagged. This reset happened because the useEffect hook queries the API every time the home page component is mounted, i.e. we overwrite the tagged movies information. We will fix this issue in the next lab - I promise!

Commit this work to the repository:

```
$ git add -A
$ git commit -m "Add to favourites logic completed; Dummy favourites page."
$ git push origin master
```

~~~{0}navigationundefined./img/navigation.pngundefined

[favorites]: ./img/favorites.png