# CSS-in-JS Pattern

Date: 2021-08-03

## Author

Mike Fisher (@mikefisher84)

## Status

Approved

## Context and Background

In today's React ecosystem most React and React Native apps are written using some sort of CSS-in-JS solution. Previously we simply used a `styles.ts` file that lived next to its related component. We've since moved towards a different pattern using a tool called Styled Components.

Styled Components extends a built-in element and allows you to attach browser-formatted CSS to it. It then uses its built-in `CSS-to-React-Native` library to transform the browser-based CSS properties
to react native-specific ones.

Because we're using `stylelint-config-react-native-styled-components` we have linting that prevents someone from using incompatible CSS properties.

### Scope of change

Write all future CSS with `styled-components`. Slowly migrate the current CSS to the Styled Components pattern.

### Wait what about Emotion?

[Emotion](https://emotion.sh/docs/introduction) is another popular CSS in JS library. Its API is nearly identical to that of Styled Components. Upon searching for resources comparing Styled Components to Emotion we found many articles and tweets from high-profile developers saying that Emotion was better than Styled Components because of certain features that Styled Components lacked.
However, most of this content is about 2+ years old. Since then Styled Components has reached v5 which now has feature parity with Emotion more or less. Currently, the only perceivable difference between Styled Components and Emotion is that Styled Components seems to have a larger community around it than Emotion.

Because of this and the fact that our original POC used Styled Components, we decided to continue down that path.

### Styled Components Example

Below we're simply styling a text element. Note how we're renaming it to something more descriptive based on what the component is actually doing, as opposed to simply using a name like `StyledText`.

```tsx
const MessageText = styled.Text`
  font-family: ${FontFamilies.sans.bold};
  color: ${(props) => props.theme.baseTheme.colors.chestnut};
  text-align: center;
  flex: 3;
`;
```

Here we're taking the `MainButton` component and extending its styles. `MainButton` already contains styles but they're specific to the component itself (color, font family, border etc). Then when we use the component we extend the styles to add additional layout styling specific to how it's being used in this particular view.
Also, note how we're renaming it to be more descriptive of what the button does in its view.

```tsx
const ButtonUpdateQuantity = styled(MainButton)`
  flex: 3;
  margin-left: ${spacePx.sm};
`;
```

Link to related PR:
[Add Styled Components](https://github.com/*redacted*/*redacted-app-name*/pull/260)
