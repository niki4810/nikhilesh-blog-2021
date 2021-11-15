---
title: 'Building a Carousel In React using Typescript'
summary: "In this blog post, I'd like to share my experience in building a basic carousel using react, hooks, typescript and some basic css."
date: 2021-11-14 17:26
tags: ['react', 'TypeScript', 'hooks', 'carousel', 'css']
draft: true
---

## Summary

I recently published a npm package called [basic-carousel](https://github.com/niki4810/basic-carousel) as a fun learning project. In this blog I'd like to share my experience in building a basic carousel using react, hooks, TypeScript and some basic css. The npm package above has a bit more feature, for this blog I will be covering the core functionality for simplicity.

The end result of it would be a carousel that looks like this

<img src="/static/images/bc/0-bc.gif" alt="basic carousel" />

> If you'd like to skip reading all of this and use the component head over to [basic-carousel](https://github.com/niki4810/basic-carousel)

## Setting the requirements

I wanted to build a basic carousel react component with the following requirements:

- It accepts tiles as children. Where each tile can be ReactNode.
- It automatically calculates how many tiles to display based on the viewport width
- It has a pagination buttons to horizontally scroll forward or backwards
- It has pagination dots as well to scroll forward and backwards

## Getting started

To get started lets bootstrap a simple react application using create react app using typescript template.

```shell
npx create-react-app my-app --template typescript
```

Once the app is bootstrapped, delete all the contents of `App.css` and have `App.tsx` return an empty `div` for now.

```javascript
import React from 'react'
import './App.css'

function App() {
  return <div></div>
}

export default App
```

For the post all of our code will be our `App.tsx` file and all the styles will be part of `App.css` for simplicity.

## Rendering the basic layout

Lets create a `BasicCarousel` React functional component and have it return the following `jsx`.

```javascript
type BasicCarouselProps = {
  children: ReactNode,
}

const BasicCarousel: React.FC<BasicCarouselProps> = ({ children }) => {
  const displayNext = true;
  const displayPrev = true;
  const displayPaginationDots = true;
  const scrollPoints = [0];
  const currentSlideIndex = 0;

  const transformedChildren = React.Children.map(children, (child, i) => {
    return (
      <div className="basic_carousel__tile" data-tile-id={i} key={`tile-${i}`}>
        {child}
      </div>
    )
  })

  /* Render */
  return (
    <div className="basic_carousel__viewport">
      <div className="basic_carousel__viewport--inner">
        <div className={`basic_carousel__btn--container`}>
          <button
            className={`basic_carousel__btn basic_carousel__btn--prev ${
              displayPrev ? 'basic_carousel__btn--active' : ''
            }`}
            onClick={() => {}}
          >
            &lt;
          </button>
        </div>
        <div className="basic_carousel__container">{transformedChildren}</div>
        <div className={`basic_carousel__btn--container`}>
          <button
            className={`basic_carousel__btn basic_carousel__btn--next ${
              displayNext ? 'basic_carousel__btn--active' : ''
            }`}
            onClick={() => {}}
          >
            &gt;
          </button>
        </div>
      </div>
      {displayPaginationDots && (
        <div className="basic_carousel__dots--container">
          {scrollPoints.map((_: number, i: number) => {
            const paginationDotsClass = `basic_carousel__dots ${
            currentSlideIndex === i ? "basic_carousel__dots--active" : ""
          }`;
            return (
              <button
                key={`slide-pagination-dot-${i}`}
                aria-label={`slide ${i}`}
                className={paginationDotsClass}
                onClick={() => {}}
              ></button>
            )
          })}
        </div>
      )}
    </div>
  )
}
```

Ignore the `const` variables and focus of two main portions of the component. First, we iterate over passed in children prop and wrap it in a div, this allows us to add custom attributes to each passed in tile element without affecting the tile element itself.

Next, we render a basic layout for our carousel. For now `displayPrev`, `displayNext` and `displayPaginationDots` are all set to true, we will add some react state to hide and show them later.

Now lets copy paste styles into `App.css`

```css
:root {
  --basic-carousel-tile-spacing: 10px;
  --basic-carousel-pagination-button-width: 35px;
  --basic-carousel-pagination-button-height: 35px;
  --basic-carousel-pagination-button-padding: 5px;
  --basic-carousel-pagination-button-color: #212121;
  --basic-carousel-pagination-button-background: #ffffff;
  --basic-carousel-pagination-button-border-color: #212121;
  --basic-carousel-pagination-dots-background: #d6d7d9;
  --basic-carousel-pagination-dots-active-background: #212121;
}

/* View Port */
.basic_carousel__viewport {
  position: relative;
  margin: 0 auto;
  box-sizing: border-box;
}
.basic_carousel__viewport:before,
.basic_carousel__viewport:after {
  box-sizing: inherit;
}

/* View Port Inner */
.basic_carousel__viewport--inner {
  display: flex;
}
/* Main Carousel */
.basic_carousel__container {
  display: flex;
  overflow-x: auto;
  overflow-y: hidden;
  scroll-snap-type: x mandatory;
}

.basic_carousel__container::-webkit-scrollbar {
  display: none;
}

/* Carousel Children */
.basic_carousel__tile {
  margin-right: var(--basic-carousel-tile-spacing);
  display: flex;
  max-width: 100%;
}

.basic_carousel__tile:last-child {
  margin-right: 0;
}

/* Pagination Buttons */
.basic_carousel__btn--container {
  min-width: var(--basic-carousel-pagination-button-width);
  padding: var(--basic-carousel-pagination-button-padding);
  display: flex;
  align-items: center;
}

.basic_carousel__btn--container.basic_carousel__btn--container-no-width {
  min-width: 0;
  padding: 0;
}

.basic_carousel__btn {
  display: none;
  font-weight: 600;
  width: var(--basic-carousel-pagination-button-width);
  height: var(--basic-carousel-pagination-button-height);
  line-height: 1;
  cursor: pointer;
  border-radius: 40px;
  border: 1px solid var(--basic-carousel-pagination-button-border-color);
  color: var(--basic-carousel-pagination-button-color);
  background: var(--basic-carousel-pagination-button-background);
}

.basic_carousel__btn.basic_carousel__btn--active {
  display: block;
}

/* Pagination dots */
.basic_carousel__dots--container {
  padding: 10px;
  display: flex;
  justify-content: center;
}
.basic_carousel__dots {
  width: 8px;
  height: 8px;
  border-radius: 8px;
  margin: 5px;
  padding: 0;
  cursor: pointer;
  border: 0;
  background: var(--basic-carousel-pagination-dots-background);
}

.basic_carousel__dots--active {
  background: var(--basic-carousel-pagination-dots-active-background);
}
```

The main css class to look for here is `.basic_carousel__container`. This is the container in which all the child tile elements are rendered, we use `flexbox` to horizontally layout all of it children set `overflow-x: auto;` allowing it to scroll horizontally and finally (most importantly) set `scroll-snap-type: x mandatory;` we will later programmatically set `scroll-snap-align` on certain child tiles so that as we scroll the carousel, the tile elements get scrolled and snapped at the correct point.

The next thing to note is each carousel tile, by default gets a `10px` left margin (see `.basic_carousel__tile` class), this allows spacing between each carousel tile.

Then render a BasicCarousel inside `App` component with some mock tile children

```javascript
function App() {
  const tiles = new Array(26).fill('').map((_, i) => `Tile - ${i}`)

  return (
    <BasicCarousel>
      {tiles.map((tile, i) => {
        return (
          <div
            key={tile}
            style={{
              width: '250px',
              height: '350px',
              background: '#dce4ef',
              color: '#212121',
              border: '5px dashed #212121',
              display: 'flex',
              alignItems: 'center',
              justifyContent: 'center',
              fontWeight: 'bold',
              fontSize: '28px',
            }}
          >
            {tile}
          </div>
        )
      })}
    </BasicCarousel>
  )
}
```

At the end of this, your app component should render something as shown below, its not functional yet, but that will be covered in the next section.

<img src="/static/images/bc/1-bc.png" alt="basic carousel 1" />

## Adding state to the carousel

Let's create a new type definition for our carousel state

```javascript
export type BasicCarouselState = {
  currentSlideIndex: number,
  totalNoOfSlides: number,
  scrollPoints: number[],
  isScrolling: boolean,
}
```

All the state variables are pretty self explanatory, except the `scrollPoints`. It will be clear what that is used for in a bit, for now its just an number array.

Next create a hook for calculating our carousel state and set the initial state

```javascript
export const useCarouselStateCalculator = (
  carouselContainerRef: React.RefObject<HTMLDivElement>
) => {
  const [carouselState, setCarouselState] =
    useState <
    BasicCarouselState >
    (() => {
      return {
        currentSlideIndex: 0,
        totalNoOfSlides: 1,
        scrollPoints: [],
        isScrolling: false,
      }
    })

  // TODO add a useEffect

  return {
    carouselState,
    setCarouselState,
  }
}
```

next add a useEffect (replace the todo comment in above snippet) that calculates our state

```javascript

useEffect(() => {
    function calculateSlides() {
      if (!carouselContainerRef.current) return;
      const CAROUSEL_OFFSET_MARGIN = 10; 
      const carouselContainerWidth =
        carouselContainerRef.current.getBoundingClientRect().width +
        CAROUSEL_OFFSET_MARGIN;
      const carouselChildren = carouselContainerRef.current?.children;

      let totalNoOfSlides = 0;
      let totalWidthSoFar = 0;
      let scrollPoints: number[] = [0];
      carouselChildren[0].setAttribute("style", "scroll-snap-align:start;");

      for (let i = 0; i < carouselChildren.length; i++) {
        const child = carouselChildren[i] as HTMLElement;
        const childWidth = child.getBoundingClientRect().width;
        totalWidthSoFar += childWidth + CAROUSEL_OFFSET_MARGIN;
        if (totalWidthSoFar >= carouselContainerWidth) {
          totalWidthSoFar = 0;
          totalWidthSoFar = totalWidthSoFar + childWidth;
          if (i !== 0) {
            scrollPoints.push(i);
          }
          child.setAttribute("style", "scroll-snap-align:start;");
        }
      }
      totalNoOfSlides = scrollPoints.length;
      setCarouselState((prevState) => {
        return {
          ...prevState,
          totalNoOfSlides,
          scrollPoints,
        };
      });

      const carouselTile = carouselContainerRef.current
        .children[0] as HTMLElement;
      if (typeof carouselContainerRef.current.scrollTo === "function") {
        carouselContainerRef.current.scrollTo({
          left: carouselTile.offsetLeft ?? 0,
          behavior: "smooth",
        });
      }
    }

    calculateSlides();

  }, [carouselContainerRef]);
```

A lot is happening in this hook, lets try to go though it step by step, by calling the calculateSlides nested function:

1. We get the `carouselContainerRef` and calculate the total carousel container viewport width and store it in a `carouselContainerWidth` variable. Notice we have added a `CAROUSEL_OFFSET_MARGIN`, its because each carousel tile has a `10px` left margin.

2. Next we initialize scrollPoints and push the first tile index `0` into it. Because initially the carousel starts from slide one and the first tile of the first slide will start at index `0`.

3. We also set `scroll-snap-align:start;` on the first tile because we want that to be our starting scroll point

4. Next we loop through each carousel child and calculate width up until that child, if at any point the totalWidth of children exceeds the carousel container width, its an indication that the tile is getting cropped from the viewport. It is at this point we push that tile index into scrollPoints array, set `scroll-snap-align:start;` indicating that its our next scroll destination. 
We do this for all the children of the carouselContainerRef.

5. By the end of this, we will have all the scroll indexes of carousel in `scrollPoints` array where each array element represents the tile index and the array index represent the slide index.

The `totalNoOfSlides` will be equal to the length of `scrollPoints` array We take all this info and set out state.


## Setting up a resize listener

There is one thing left in our above hook, we want our carousel to adjust its tiles and slide indexes when users resize the browser, for this we setup a resize listener, but debounce it for performance reasons.

```javascript
    let timeout: number;
    /* istanbul ignore next */
    function handleResize() {
      if (timeout) clearTimeout(timeout);
      timeout = setTimeout(() => {
        calculateSlides();
      }, 300) as unknown as number;
    }

    calculateSlides();

    window.addEventListener("resize", handleResize);
    return () => {
      window.removeEventListener("resize", handleResize);
    };

```

Our total hook looks like this:

```javascript
export const useCarouselStateCalculator = (carouselContainerRef: React.RefObject<HTMLDivElement>) => {
  const [carouselState, setCarouselState] = useState<BasicCarouselState>(() => {
    return {
      currentSlideIndex: 0,
      totalNoOfSlides: 1,
      scrollPoints: [],
      isScrolling: false,
    };
  });

  useEffect(() => {
    function calculateSlides() {
      if (!carouselContainerRef.current) return;
      const CAROUSEL_OFFSET_MARGIN = 10;
      const carouselContainerWidth =
        carouselContainerRef.current.getBoundingClientRect().width +
        CAROUSEL_OFFSET_MARGIN;
      const carouselChildren = carouselContainerRef.current?.children;

      let totalNoOfSlides = 0;
      let totalWidthSoFar = 0;
      let scrollPoints: number[] = [0];
      carouselChildren[0].setAttribute("style", "scroll-snap-align:start;");

      for (let i = 0; i < carouselChildren.length; i++) {
        const child = carouselChildren[i] as HTMLElement;
        const childWidth = child.getBoundingClientRect().width;
        totalWidthSoFar += childWidth + CAROUSEL_OFFSET_MARGIN;
        if (totalWidthSoFar >= carouselContainerWidth) {
          totalWidthSoFar = 0;
          totalWidthSoFar = totalWidthSoFar + childWidth;
          if (i !== 0) {
            scrollPoints.push(i);
          }
          child.setAttribute("style", "scroll-snap-align:start;");
        }
      }
      totalNoOfSlides = scrollPoints.length;
      setCarouselState((prevState) => {
        return {
          ...prevState,
          totalNoOfSlides,
          scrollPoints,
        };
      });

      const carouselTile = carouselContainerRef.current
        .children[0] as HTMLElement;
      if (typeof carouselContainerRef.current.scrollTo === "function") {
        carouselContainerRef.current.scrollTo({
          left: carouselTile.offsetLeft ?? 0,
          behavior: "smooth",
        });
      }
    }

    /* On resize recalculate carousel slides */
    let timeout: number;
    /* istanbul ignore next */
    function handleResize() {
      if (timeout) clearTimeout(timeout);
      timeout = setTimeout(() => {
        calculateSlides();
      }, 300) as unknown as number;
    }

    calculateSlides();

    window.addEventListener("resize", handleResize);
    return () => {
      window.removeEventListener("resize", handleResize);
    };
  }, [carouselContainerRef]);

  return {
    carouselState,
    setCarouselState,
  };
};
```

## Combine our hook to view

Our state calculation hook is now complete. the last thing left to do is tie it up to over component and adjust a few things. To get started, within our `BasicCarousel` component:

1. Create a ref for `carouselContainerRef`

```javascript
const carouselContainerRef = useRef<HTMLDivElement>(null);
```

2. Call out `useCarouselStateCalculator` hook and pass the `carouselContainerRef`

```javascript
const { carouselState, setCarouselState } = useCarouselStateCalculator(carouselContainerRef);
```

3. Replace the const variables with this, this basically calculates the pagination display variables from state.

```javascript
  const { totalNoOfSlides, currentSlideIndex, scrollPoints } = carouselState;
  const displayPrev = totalNoOfSlides > 1 && currentSlideIndex > 0;
  const displayNext = totalNoOfSlides > 1 && currentSlideIndex + 1 !== totalNoOfSlides;
  const displayPaginationDots = scrollPoints.length > 1;
```

4. Create a few event handlers for navigating forward and backwards as we click the pagination buttons

```javascript
/* Helpers */
  const navigateToSlideBySlideIndex = (slideIndex: number) => {
    const { scrollPoints, currentSlideIndex } = carouselState;
    const nextTileIndex = scrollPoints[slideIndex];
    if (nextTileIndex !== null && nextTileIndex !== undefined) {
      const carouselTile = carouselContainerRef.current?.children[
        nextTileIndex
      ] as HTMLElement;

      const parentEl = carouselTile.offsetParent as HTMLElement;

      if (typeof carouselContainerRef.current?.scrollTo === "function") {
        carouselContainerRef.current?.scrollTo({
          left: carouselTile.offsetLeft - (parentEl?.offsetLeft ?? 0),
          behavior: "smooth",
        });
      }

      if (currentSlideIndex !== slideIndex) {
        setCarouselState((prevState) => {
          return {
            ...prevState,
            currentSlideIndex: slideIndex,
          };
        });
      }
    }
  };

  const navigateSlides = (direction: "forward" | "backwards") => {
    const currentSlideIndex = carouselState.currentSlideIndex;
    const nextSlideIndex =
      direction === "forward" ? currentSlideIndex + 1 : currentSlideIndex - 1;
    navigateToSlideBySlideIndex(nextSlideIndex);
  };

  /* Event Handlers */
  const handlePrevClick = () => {
    navigateSlides("backwards");
  };

  const handleNextClick = () => {
    navigateSlides("forward");
  };

  const handleSlideClick = (slideIndex: number) => {
    navigateToSlideBySlideIndex(slideIndex);
  };
```

5. Attach the `ref={carouselContainerRef}` to the div with className of `basic_carousel__container`

```javascript
<div className="basic_carousel__container" ref={carouselContainerRef}>
```

6. Attach onClick callback for pagination buttons

```javascript
    <button
            className={`basic_carousel__btn basic_carousel__btn--prev ${
              displayPrev ? "basic_carousel__btn--active" : ""
            }`}
            onClick={handlePrevClick}
          >
            &lt;
          </button>
```

```javascript
<button
            className={`basic_carousel__btn basic_carousel__btn--next ${
              displayNext ? "basic_carousel__btn--active" : ""
            }`}
            onClick={handleNextClick}
          >
            &gt;
          </button>
```

```javascript
 <button
    key={`slide-pagination-dot-${i}`}
    aria-label={`slide ${i}`}
    className={paginationDotsClass}
    onClick={() => {
        handleSlideClick(i);
    }}
    ></button>
```

This is pretty straight forward react code where we attach an onClick handler for each pagination button.

Finally here is the complete code for out `BasicCarousel` Component

```javascript
const BasicCarousel: React.FC<BasicCarouselProps> = ({ children }) => {
  const carouselContainerRef = useRef<HTMLDivElement>(null);
  const { carouselState, setCarouselState } = useCarouselStateCalculator(carouselContainerRef);

  const { totalNoOfSlides, currentSlideIndex, scrollPoints } = carouselState;
  const displayPrev = totalNoOfSlides > 1 && currentSlideIndex > 0;
  const displayNext = totalNoOfSlides > 1 && currentSlideIndex + 1 !== totalNoOfSlides;
  const displayPaginationDots = scrollPoints.length > 1;

  /* Helpers */
  const navigateToSlideBySlideIndex = (slideIndex: number) => {
    const { scrollPoints, currentSlideIndex } = carouselState;
    const nextTileIndex = scrollPoints[slideIndex];
    if (nextTileIndex !== null && nextTileIndex !== undefined) {
      const carouselTile = carouselContainerRef.current?.children[
        nextTileIndex
      ] as HTMLElement;

      const parentEl = carouselTile.offsetParent as HTMLElement;

      if (typeof carouselContainerRef.current?.scrollTo === "function") {
        carouselContainerRef.current?.scrollTo({
          left: carouselTile.offsetLeft - (parentEl?.offsetLeft ?? 0),
          behavior: "smooth",
        });
      }

      if (currentSlideIndex !== slideIndex) {
        setCarouselState((prevState) => {
          return {
            ...prevState,
            currentSlideIndex: slideIndex,
          };
        });
      }
    }
  };

  const navigateSlides = (direction: "forward" | "backwards") => {
    const currentSlideIndex = carouselState.currentSlideIndex;
    const nextSlideIndex =
      direction === "forward" ? currentSlideIndex + 1 : currentSlideIndex - 1;
    navigateToSlideBySlideIndex(nextSlideIndex);
  };

  /* Event Handlers */
  const handlePrevClick = () => {
    navigateSlides("backwards");
  };

  const handleNextClick = () => {
    navigateSlides("forward");
  };

  const handleSlideClick = (slideIndex: number) => {
    navigateToSlideBySlideIndex(slideIndex);
  };

  const transformedChildren = React.Children.map(children, (child, i) => {
    return (
      <div className="basic_carousel__tile" data-tile-id={i} key={`tile-${i}`}>
        {child}
      </div>
    );
  });

  /* Render */
  return (
    <div className="basic_carousel__viewport">
      <div className="basic_carousel__viewport--inner">
        <div
          className={`basic_carousel__btn--container`}
        >
          <button
            className={`basic_carousel__btn basic_carousel__btn--prev ${
              displayPrev ? "basic_carousel__btn--active" : ""
            }`}
            onClick={handlePrevClick}
          >
            &lt;
          </button>
        </div>
        <div className="basic_carousel__container" ref={carouselContainerRef}>{transformedChildren}</div>
        <div
          className={`basic_carousel__btn--container`}
        >
          <button
            className={`basic_carousel__btn basic_carousel__btn--next ${
              displayNext ? "basic_carousel__btn--active" : ""
            }`}
            onClick={handleNextClick}
          >
            &gt;
          </button>
        </div>
      </div>
      {displayPaginationDots && (
        <div
          className="basic_carousel__dots--container"
        >
          {scrollPoints.map((_: number, i: number) => {
            const paginationDotsClass = `basic_carousel__dots ${
            currentSlideIndex === i ? "basic_carousel__dots--active" : ""
            }`;
            return (
              <button
                key={`slide-pagination-dot-${i}`}
                aria-label={`slide ${i}`}
                className={paginationDotsClass}
                onClick={() => { handleSlideClick(i);}}
              ></button>
            );
          })}
        </div>
      )}
    </div>
  );
};
```

If everything goes well, we should not have a functional carousel that paginates forward and backwards and snaps at the first cropped tile as we paginate.


## Conclusion

Through this [basic-carousel](https://github.com/niki4810/basic-carousel) pet project, I personally learned a lot of stuff and had so much fun building it. The original package has a bit more code to handle edge-cases but I wanted to cover the core functionality in this blog. 

If you got this far, I hope you have a working carousel and I hope you like building it as much as I did.

Thanks for reading 🙌

