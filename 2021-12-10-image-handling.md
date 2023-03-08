# Responsive Image Handling

Date: 2021-12-10

## Author

Mike Fisher (@mikefisher84)

## Status

Approved

## Context and Background

In _redacted-app-name_, our image handling needed a bit of work. The way we were working with image urls was a mixed bag with lots of URL string mutation and various
bits of logic sprinkled throughout the app.

We use a very powerful and flexible image hosting service called "Imgix" to serve our images. The main features that it provides that are of interest
to us are passing in specific pixel values to get the correct image size, as well as passing in a `dpr` or `devicePixelRatio` to support HiDPI devices.

Our GraphQL image resolver wouldn't always return properly formatted URLs. Because of this, we had logic in various components to detect the formatting of URLs.
We would then _sometimes_ append a height and width to the URL to get the correct dimensions for a particular image component. However, in other parts of the app, we would completely ignore this and download massive images (sometimes 10MB+ in size) which really slowed down performance for certain sections of the app.
Because of this, we set out to find a proper solution that would simplify our image handling and provide the best user experience regardless of which client
is consuming it (ie web or mobile)

### Scope of change

Remove all URL detection logic from _redacted-app-name_ and move it to the Image resolver in the _redacted-app-name_. Now, _redacted-app-name_ always returns properly formatted URLS minus height, width, dpr.

- Create a `ResponsiveImage` component that is the sole component for pulling in images from Imgix (ie not intended to be used for images stored locally with the application)
- Detect the size of the component and DPI of the device by accepting height/width props and passing them into the `PixelRatio.getPixelSizeForLayoutSize()` method to get upscaled pixel values
  for the particular device the app is running on.
- Move all placeholder image handling logic to the `ResponsiveImage` component instead of being handled by a separate component.
- Handle iterating over a sequential list of components by accepting a placeholderIndex prop in `ResponsiveImage`

### Other options considered

- Image Resolver accepts height, width and dpr as arguments in the resolver.
  We seriously considered this approach because it seemed like a nice feature for our GraphQL API.
  However, upon further investigation due to component hierarchy, it seemed difficult to cleanly get the correct size
  of a nested image component and pass it up a few levels to its parent component which did the communication with GraphQL.

## Decision

Forgo accepting arguments for the Image GraphQL resolver in favor of mutating the string in MMA.
While this initially seems like a less ideal solution by only mutating the string in the `ResponsiveImage` component
we determined this was the best compromise between a clean API and having the best experience for users.
