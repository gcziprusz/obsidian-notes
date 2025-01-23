### [Kneel Diamonds ERD](https://github.com/nashville-software-school/client-side-mastery/blob/master/book-5-kneel-diamonds/chapters/KD_ERD.md)

- **Elaborate Story**: Is the detailed story necessary, or could the content be streamlined?
- **Video Hosting**: Should the Vimeo link be ported to Screencastify for consistency?
- **ERD Visualization**: Consider showing the completed ERD upfront to give students a clearer reference.
### [The Map Array Method](https://github.com/nashville-software-school/client-side-mastery/blob/master/book-5-kneel-diamonds/chapters/KD_MAP_METHOD_INTRO.md#the-map-array-method)

- **Pro Tip**: Reword for clarity—some students may find it confusing or need to reread to understand.
- **Method Anatomy**: Add a section breaking down the structure of `map`, `filter`, and `find`. This could include:
    - A detailed explanation of how `map` works.
    - Side by side equivalent 'for loop' and 'map' code ?
    - Highlight that `map` always returns a new array.
- I think we should include the forEach method because it is the closest equivalent to for loops they aleady know and seems like a good immediate step to .map etc.
### [Storing User Choices](https://github.com/nashville-software-school/client-side-mastery/blob/master/book-5-kneel-diamonds/chapters/KD_CHANGE_EVENTS.md#storing-user-choices)

- **Transient State**: Add a section explaining transient state:
    - Define what transient state is and how it differs from persistent state.
    - Explain where transient state lives (e.g., JavaScript memory in the browser).
### [Showing Prices on Jewelry Orders](https://github.com/nashville-software-school/client-side-mastery/blob/master/book-5-kneel-diamonds/chapters/KD_ORDER_PRICE.md#showing-prices-on-jewelry-orders)

- **Avoid JSON-Server Specific Features**:
    - Teaching `expand` and `embed` features of `json-server` may not be worthwhile since students won’t use these features outside of NSS or even once they get to a real backend.
    - Consider teaching how to make subsequent API calls to retrieve related resources, which is more reflective of real-world practices.