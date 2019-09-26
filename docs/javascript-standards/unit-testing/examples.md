---
id: unit-testing-examples
title: Examples
---

## Node

### app.js

```javascript
const express = require("express");
const app = express();
app.get("/dog", (req, res) => res.send("Woof"));
app.get("/cat", (req, res) => res.send("Meow"));
app.get("/wookie", (req, res) => res.send("Wuuah"));
module.exports = app;
```

### app.test.js

```javascript
const request = require("supertest");
const app = require("./app");
describe("Node App", () => {
  it("Should get dog sound", async () => {
    const response = await request(app).get("/dog");
    expect(response.text).toBe("Woof");
  });
  it("Should get car sound", async () => {
    const response = await request(app).get("/cat");
    expect(response.text).toBe("Meow");
  });
  it("Should get wookie sound", async () => {
    const response = await request(app).get("/wookie");
    expect(response.text).toBe("Wuuah");
  });
});
```

### server.js

```javascript
const app = require("app");
const port = 3000;
app.listen(port, () => console.log(`Example app listening on port ${port}!`));
```

## React

### Counter.js

```jsx
import React, { useState } from "react";
import PropTypes from "prop-types";
import styled from "styled-components";

const Display = styled.div`
  font-size: 2rem;
  color: ${p => (p.currentNumber > 5 ? "red" : "black")};
  text-align: center;
`;

const Button = styled.button`
  padding: 5px 15px;
`;

const Counter = ({ start }) => {
  const [number, setNumber] = useState(Math.max(start, 0));

  const increaseCount = () => {
    setNumber(number + 1);
  };

  const decreaseCount = () => {
    const decreasedNumber = Math.max(number - 1, 0);
    setNumber(decreasedNumber);
  };

  return (
    <div>
      <Button data-cv-test="minus-button" onClick={decreaseCount}>
        -
      </Button>
      <Display currentNumber={number}>{number}</Display>
      <Button data-cv-test="plus-button" onClick={increaseCount}>
        +
      </Button>
    </div>
  );
};

Counter.defaultProps = {
  start: 1
};

Counter.propTypes = {
  start: PropTypes.number
};

export default Counter;
```

#### Counter.test.js

```jsx
import React from "react";
import { render, fireEvent } from "@testing-library/react";
import Counter from "./Counter";

describe("Counter", () => {
  const renderCounter = (props = {}) => render(<Counter {...props} />);

  it("Should render numbers differently over five", () => {
    const { container } = renderCounter({ start: 6 });
    expect(container).toMatchSnapshot();
  });

  it("Should start at one", () => {
    const { getByText, container } = renderCounter();
    expect(getByText("1")).toBeDefined();
    expect(container).toMatchSnapshot();
  });

  it("Should increase number on button push", () => {
    const { container, getByText } = renderCounter();
    fireEvent.click(container.querySelector("[data-cv-test=plus-button]"));
    expect(getByText("2")).toBeDefined();
  });

  it("Should decrease count on button push", () => {
    const { container, getByText } = renderCounter({ start: 3 });
    fireEvent.click(container.querySelector("[data-cv-test=minus-button]"));
    expect(getByText("2")).toBeDefined();
  });

  it("Should not allow a start number below zero", () => {
    const { getByText } = renderCounter({ start: -1 });
    expect(getByText("0")).toBeDefined();
  });

  it("Should not user to decrease below zero", () => {
    const { getByText, container } = renderCounter({ start: 0 });
    fireEvent.click(container.querySelector("[data-cv-test=minus-button]"));
    expect(getByText("0")).toBeDefined();
  });
});
```

## Redux

### actions.js

```js
import * as types from "./actionTypes";
import { updateFiltersByType, FilterSource } from "../../Facets/actions";
export const updateFilters = payload => (dispatch, getState) => {
  const {
    facets: { data: facetData }
  } = getState();
  const { key } = payload;
  let { category } = payload;
  let facet;
  if (category === "models") {
    const { encodedId, makeName, modelName } = payload;
    category = "make_model";
    facet = facetData.makes.find(x => x.displayName === makeName)
      ? facetData.makes
          .find(x => x.displayName === makeName)
          .models.find(
            x => x.encodedId === encodedId && x.displayName === modelName
          )
      : null;
  } else if (category === "isFreeDelivery") {
    facet = facetData[category];
  } else {
    facet = facetData[category].find(x => x.key === key);
  }
  if (!facet || !category) {
    return dispatch({ type: "UPDATE_FILTERS_FROM_SUGGESTED_FILTER_FAILED" });
  }
  dispatch({
    type: "UPDATE_FILTERS_FROM_SUGGESTED_FILTER"
  });
  return dispatch(
    updateFiltersByType(category, facet, true, FilterSource.Suggested)
  );
};
```

### actions.test.js

```javascript
import { updateFilters } from "./actions";
import * as facetActions from "../../Facets/actions";
describe("Suggested Filters actions", () => {
  facetActions.updateFiltersByType = jest.fn();
  const dispatch = jest.fn();
  const pickupFacet = { bitMask: 4, key: "Pickup", displayName: "Truck" };
  const audiA7Facet = { encodedId: "e1", displayName: "A7" };
  const isFreeDelivery = { displayName: "FreeShipping" };
  const getState = () => ({
    facets: {
      data: {
        bodyStyles: [pickupFacet],
        makes: [
          {
            displayName: "Audi",
            models: [audiA7Facet]
          }
        ],
        isFreeDelivery
      }
    }
  });
  beforeEach(() => {
    dispatch.mockClear();
    facetActions.updateFiltersByType.mockClear();
  });
  it("Should dispatch update filters by type", () => {
    const payload = {
      key: "Pickup",
      category: "bodyStyles"
    };
    updateFilters(payload)(dispatch, getState);
    expect(dispatch).toHaveBeenCalledTimes(2);
    expect(dispatch).toHaveBeenCalledWith({
      type: "UPDATE_FILTERS_FROM_SUGGESTED_FILTER"
    });
    expect(facetActions.updateFiltersByType).toHaveBeenCalledWith(
      payload.category,
      pickupFacet,
      true,
      facetActions.FilterSource.Suggested
    );
  });
  it("Should find model in facet data and update filters", () => {
    const payload = {
      encodedId: "e1",
      makeName: "Audi",
      modelName: "A7",
      category: "models"
    };
    updateFilters(payload)(dispatch, getState);
    expect(dispatch).toHaveBeenCalledTimes(2);
    expect(facetActions.updateFiltersByType).toHaveBeenCalledWith(
      "make_model",
      audiA7Facet,
      true,
      facetActions.FilterSource.Suggested
    );
  });
  it("Should fail gracefully if make cannot be found", () => {
    const payload = {
      encodedId: "a1",
      makeName: "Honda",
      modelName: "Accord",
      category: "models"
    };
    updateFilters(payload)(dispatch, getState);
    expect(dispatch).toHaveBeenLastCalledWith({
      type: "UPDATE_FILTERS_FROM_SUGGESTED_FILTER_FAILED"
    });
  });
  it("Should handle free delivery selection", () => {
    const payload = {
      category: "isFreeDelivery"
    };
    updateFilters(payload)(dispatch, getState);
    expect(facetActions.updateFiltersByType).toHaveBeenCalledWith(
      "isFreeDelivery",
      isFreeDelivery,
      true,
      facetActions.FilterSource.Suggested
    );
  });
});
```

### ConnectedComponent.js

```jsx
import React, { useRef } from "react";
import PropTypes from "prop-types";
import { bindActionCreators } from "redux";
import { connect } from "react-redux";
import { compose, onlyUpdateForKeys } from "recompose";
import ArrowBackIcon from "../../../assets/svgs/icons/ArrowBack";
import ArrowForwardIcon from "../../../assets/svgs/icons/ArrowForward";
import {
  SuggestedFiltersWrapper,
  FiltersWrapper,
  LoadingSuggestedFiltersButton,
  ShowPrevButton,
  ShowNextButton,
  SuggesterFilterButton
} from "../SuggestedFilters.styles";
import * as actions from "./actions";
import theme from "../../../theme";
import formatNum from "../../../utilities/formatNumWithCommas";

export const SuggestedFilters = ({
  isLoading,
  suggestedFilters,
  screenSize,
  updateFilters
}) => {
  const filtersWrapper = useRef(null);

  const handleScrollPrevPage = () => {
    const containerWidth = filtersWrapper.current.getBoundingClientRect().width;
    filtersWrapper.current.scrollBy({
      left: -(containerWidth / 2),
      behavior: "smooth"
    });
  };

  const handleScrollNextPage = () => {
    const containerWidth = filtersWrapper.current.getBoundingClientRect().width;
    filtersWrapper.current.scrollBy({
      left: containerWidth / 2,
      behavior: "smooth"
    });
  };

  return isLoading ? (
    <SuggestedFiltersWrapper
      isMobile={screenSize.isMobile}
      isLoading={isLoading}
    >
      <FiltersWrapper>
        {Array.from(Array(20).keys()).map(x => (
          <LoadingSuggestedFiltersButton key={x} />
        ))}
      </FiltersWrapper>
    </SuggestedFiltersWrapper>
  ) : (
    <SuggestedFiltersWrapper
      data-cv-test="Cv.Search.SuggestedFilters.Container"
      isLoading={isLoading}
    >
      {screenSize.width > theme.breakpoints.values.sm &&
        suggestedFilters.length > 0 && (
          <ShowPrevButton
            onClick={handleScrollPrevPage}
            size="small"
            variant="flat"
            data-cv-test="sf-prev-button"
          >
            <ArrowBackIcon />
          </ShowPrevButton>
        )}
      <FiltersWrapper innerRef={filtersWrapper}>
        {suggestedFilters.map(filter => (
          <SuggesterFilterButton
            key={`${filter.category}${filter.displayName}`}
            className="filter"
            onClick={() => updateFilters(filter)}
            data-cv-test={`suggestedFilter.${filter.displayName}`}
          >
            {filter.displayName} ({formatNum(filter.totalCount)})
          </SuggesterFilterButton>
        ))}
      </FiltersWrapper>
      {screenSize.width > theme.breakpoints.values.sm &&
        suggestedFilters.length > 0 && (
          <ShowNextButton
            onClick={handleScrollNextPage}
            size="small"
            variant="flat"
            data-cv-test="sf-next-button"
          >
            <ArrowForwardIcon />
          </ShowNextButton>
        )}
    </SuggestedFiltersWrapper>
  );
};

SuggestedFilters.defaultProps = {
  suggestedFilters: []
};

SuggestedFilters.propTypes = {
  screenSize: PropTypes.shape({}).isRequired,
  isLoading: PropTypes.bool.isRequired,
  suggestedFilters: PropTypes.arrayOf(PropTypes.shape({})),
  updateFilters: PropTypes.func.isRequired
};

const mapStateToProps = state => ({
  isLoading: state.suggestedFilters.isLoading,
  suggestedFilters: state.suggestedFilters.data,
  screenSize: state.ui.screenSize
});

const mapDispatchToProps = dispatch =>
  bindActionCreators({ ...actions }, dispatch);

export default compose(
  connect(
    mapStateToProps,
    mapDispatchToProps
  ),
  onlyUpdateForKeys(["isLoading", "screenSize"])
)(SuggestedFilters);
```

### ConnectedComponent.test.js

```jsx
import React from "react";
import { ThemeProvider } from "@carvana/theme";
import { render, fireEvent } from "@testing-library/react";
import { SuggestedFilters } from "./SuggestedFilters";

describe("SuggestedFilters", () => {
  const updateFilters = jest.fn();

  const observeMock = {
    observe: () => null,
    unobserve: () => null
  };

  beforeEach(() => {
    window.IntersectionObserver = () => observeMock;
    updateFilters.mockClear();
  });

  const screenSize = {
    width: 1024,
    isMobile: false
  };
  const suggestedFilters = [];

  const filterData = [
    {
      displayName: "Truck",
      category: "bodyStyles",
      totalCount: "100",
      score: 0.08,
      isSelected: false,
      key: "Pickup"
    },
    {
      displayName: "Sun Roof",
      category: "features",
      totalCount: "50",
      score: 0.05,
      isSelected: false,
      key: "SunRoof"
    }
  ];

  const initialProps = {
    screenSize,
    isLoading: true,
    suggestedFilters,
    updateFilters
  };

  const loadedProps = {
    screenSize,
    isLoading: false,
    suggestedFilters: filterData,
    updateFilters
  };

  const renderComponent = (props, wrapper) =>
    render(
      <ThemeProvider>
        <SuggestedFilters {...props} />
      </ThemeProvider>,
      wrapper
    );

  it("Should render 20 placeholder blocks", () => {
    const { container } = renderComponent(initialProps);
    expect(container.querySelectorAll("button").length).toBe(20);
  });

  it("Should render suggested filters with counts", () => {
    const { container, getByText } = renderComponent(loadedProps);
    expect(container.querySelectorAll("button").length).toBe(2);
    expect(getByText("Sun Roof (50)")).toBeDefined();
  });

  it("Should call update filters on click", () => {
    const { container } = renderComponent(loadedProps);
    fireEvent.click(container.querySelector("button"));
    expect(updateFilters).toHaveBeenCalledTimes(1);
  });

  it("Should render nothing if no suggested filters are provided", () => {
    const { container } = renderComponent({
      ...initialProps,
      isLoading: false
    });
    expect(container.querySelectorAll("button").length).toBe(0);
  });
});
```

### reducers.js

```js
import initialState from "../../../store/initialState";
import * as types from "./actionTypes";
import { FETCH_INVENTORY } from "../../Search/actionTypes";

export default function suggestedFilters(
  state = initialState.suggestedFilters,
  action
) {
  const { type, payload } = action;
  const isError = type.includes("_ERROR");

  switch (type) {
    case `${FETCH_INVENTORY}_LOADING`:
    case `${types.FETCH_SUGGESTED_FILTERS}_LOADING`:
      return {
        ...state,
        isLoading: true
      };
    case `${types.FETCH_SUGGESTED_FILTERS}_ERROR`:
    case `${types.FETCH_SUGGESTED_FILTERS}_SUCCESS`:
      return {
        ...state,
        data: isError ? [] : payload,
        isLoading: false,
        error: isError ? payload : null,
        isError
      };
    default:
      return state;
  }
}
```

### reducers.test.js

```js
import * as types from "./actionTypes";
import suggestedFilters from "./reducers";
import { FETCH_INVENTORY } from "../../Search/actionTypes";

describe("Suggested Filter Reducer", () => {
  const initialState = {
    data: [],
    isLoading: true
  };

  it("Should return default if no action is fired", () => {
    const result = suggestedFilters(initialState, { type: "" });
    expect(result).toEqual(initialState);
  });

  it("Should be in loading state during suggested filter load", () => {
    const result = suggestedFilters(initialState, {
      type: `${types.FETCH_SUGGESTED_FILTERS}_LOADING`
    });
    expect(result).toEqual(initialState);
  });

  it("Should be in loading state during inventory load", () => {
    const result = suggestedFilters(initialState, {
      type: `${FETCH_INVENTORY}_LOADING`
    });
    expect(result).toEqual(initialState);
  });

  it("Should return successful state when fetch is successful", () => {
    const fetchedFilters = ["filter"];
    const result = suggestedFilters(initialState, {
      type: `${types.FETCH_SUGGESTED_FILTERS}_SUCCESS`,
      payload: fetchedFilters
    });
    expect(result).toEqual({
      isLoading: false,
      data: fetchedFilters,
      error: null,
      isError: false
    });
  });

  it("Should return error state when fetch has error", () => {
    const error = "error";
    const result = suggestedFilters(initialState, {
      type: `${types.FETCH_SUGGESTED_FILTERS}_ERROR`,
      payload: error
    });
    expect(result).toEqual({
      isLoading: false,
      data: [],
      error,
      isError: true
    });
  });
});
```

### selectors.js

```javascript
import createSelector from "../../utilities/createSelector";

export const getPickupInfoFromDeliveryContext = createSelector(
  [
    state =>
      (state.user.deliveryContext &&
        state.user.deliveryContext.pickUpInfoToDisplay) ||
      {}
  ],
  pickupLocation => pickupLocation
);
```

### selectors.test.js

```js
import { getPickupInfoFromDeliveryContext } from "../reducers";

describe("Selectors", () => {
  it("Should return empty object if deliveryContext is not populated", () => {
    const state = {
      user: {
        deliveryContext: {}
      }
    };
    const pickupLocation = getPickupInfoFromDeliveryContext(state);
    expect(pickupLocation).toEqual({});
  });

  it("Should return pickup location", () => {
    const state = {
      user: {
        deliveryContext: {
          pickUpInfoToDisplay: "pickup"
        }
      }
    };
    const pickupLocation = getPickupInfoFromDeliveryContext(state);
    expect(pickupLocation).toEqual("pickup");
  });
});
```
