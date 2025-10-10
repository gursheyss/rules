# Unistyles v3 Guidelines

**Always use Unistyles v3 syntax. The v2 API (createStyleSheet, useStyles) is deprecated.**

## 0\. Core Syntax Rules (v3)

Use the new v3 API exclusively.

  * **Imports:** ALWAYS import `StyleSheet` from `'react-native-unistyles'`, NOT from `'react-native'`.
      * **Correct:**
        ```typescript
        import { StyleSheet } from 'react-native-unistyles';
        ```
      * **Incorrect (v2 syntax - DO NOT USE):**
        ```typescript
        import { createStyleSheet, useStyles } from 'react-native-unistyles';
        ```
  * **Creating Styles:** Use `StyleSheet.create` directly - the styles are immediately usable without any hook.
      * **Correct:**
        ```typescript
        const styles = StyleSheet.create(theme => ({
          container: {
            backgroundColor: theme.colors.background,
          },
        }));
        
        // Use directly in JSX
        <View style={styles.container} />
        ```
      * **Incorrect (v2 syntax - DO NOT USE):**
        ```typescript
        const stylesheet = createStyleSheet(theme => ({...}));
        const { styles } = useStyles(stylesheet);
        ```
  * **Configuration:** Use `StyleSheet.configure` instead of `UnistylesRegistry`.
      * **Correct:**
        ```typescript
        import { StyleSheet } from 'react-native-unistyles';
        
        StyleSheet.configure({
          themes: { light, dark },
          breakpoints,
          settings: {
            adaptiveThemes: true,
            initialTheme: 'dark',
          }
        });
        ```
  * **No Style Spreading:** NEVER spread Unistyles styles as it removes the C++ state. Use array syntax instead.
      * **Correct:**
        ```typescript
        <View style={[styles.container, styles.primary]} />
        ```
      * **Incorrect:**
        ```typescript
        <View style={{...styles.container, ...styles.primary}} /> // Will cause errors!
        ```
  * **Variants:** Use `styles.useVariants()` method instead of passing to a hook.
      * **Correct:**
        ```typescript
        styles.useVariants({
          size: 'large',
          color: 'primary'
        });
        ```
      * **Incorrect (v2 syntax):**
        ```typescript
        const { styles } = useStyles(stylesheet, { size: 'large' });
        ```
  * **Theme Access in Components:** Use `withUnistyles` HOC to wrap components that need theme values.
      * **Example:**
        ```typescript
        import { Button } from 'react-native';
        import { withUnistyles } from 'react-native-unistyles';
        
        const UniButton = withUnistyles(Button, theme => ({
          color: theme.colors.primary
        }));
        ```
  * **Conditional Rendering by Breakpoint:** Use `Display` and `Hide` components with `mq` instead of accessing breakpoint runtime.
      * **Example:**
        ```typescript
        import { Display, Hide, mq } from 'react-native-unistyles';
        
        <Display mq={mq.only.width(0, 400)}>
          <Text>Small screens only</Text>
        </Display>
        
        <Hide mq={mq.only.width(400)}>
          <Text>Hidden on large screens</Text>
        </Hide>
        ```
  * **hairlineWidth:** Use `StyleSheet.hairlineWidth` instead of `UnistylesRuntime.hairlineWidth`.
      * **Correct:**
        ```typescript
        borderWidth: StyleSheet.hairlineWidth
        ```

-----

## 1\. Performance and Dynamic Values (No Hooks)

Avoid React hooks for styling to maintain performance.

  * **Runtime Access is Mandatory:** When styling depends on dynamic environmental values (screen size, insets, color scheme), access them via the `runtime` object passed to `StyleSheet.create`. **NEVER** use `useWindowDimensions()` or similar React hooks for this purpose.
      * **Example (Screen and Safe Area):**
        ```typescript
        const styles = StyleSheet.create({ 
          header: (theme, runtime) => ({
            paddingTop: runtime.insets.top,
            backgroundColor: theme.colors.background,
          }),
        });
        ```
  * **Keyboard Handling:** Use `runtime.insets.ime` to correctly adjust layout when the software keyboard is active (IME - Input Method Editor).
      * **Example (Footer Padding):**
        ```typescript
        const styles = StyleSheet.create({ 
          footer: (theme, runtime) => ({
            paddingBottom: runtime.insets.ime || 0, // Fallback to 0 or a fixed value
          }),
        });
        ```
  * **Prop-Based Styling:** Use **dynamic functions** for styles that depend on component props.
      * **Example (Pressed State):**
        ```typescript
        const styles = StyleSheet.create({
          button: ({ isPressed }) => ({
            opacity: isPressed ? 0.8 : 1,
          }),
        });
        ```

-----

## 2\. Theming and Configuration

Keep the theme files clean and enable automatic system switching.

  * **Atomic Theme Values Only:** The theme object (in `unistyles.ts`) **MUST** only contain atomic values: colors, single-unit spacing, and font definitions. **DO NOT** put compound component styles (e.g., a full button style) into the theme.
  * **Adaptive Theme Default:** Always configure UniStyles with `adaptiveThemes: true` to enable automatic switching between `light` and `dark` themes based on the OS.
      * **Example (Configuration):**
        ```typescript
        // unistyles.ts
        configure({ themes: { light, dark }, breakpoints, adaptiveThemes: true });
        ```

-----

## 3\. Variants and Component Design

Use variants to build flexible and reusable components.

  * **Define Variants:** Define reusable styles using the `variants` property for common style properties like `size` or `color`.
      * **Example (Basic Variant):**
        ```typescript
        variants: { 
          size: {
            sm: { fontSize: 12 },
            lg: { fontSize: 24 },
          },
        } 
        ```
  * **Use Compound Variants:** Use `compoundVariants` to apply a style when a **specific combination** of two or more variants is active.
      * **Example (Compound Style):**
        ```typescript
        compoundVariants: [
          { size: 'lg', color: 'primary', style: { fontWeight: 'bold', padding: 16 } }
        ]
        ```
  * **Type Safety:** When consuming variants in a component, use **`UniStylesVariants`** for type inference. **DO NOT** manually define the variant props.
      * **Example (Component Typing):**
        ```typescript
        import { UniStylesVariants } from 'react-native-unistyles';
        // Infer 'size' and 'color' props from the generated styles.
        type MyTextProps = TextProps & UniStylesVariants<typeof styles>; 
        ```

-----

## 4\. Web and Third-Party Interoperability

Handle cross-platform differences gracefully.

  * **Web-Only CSS:** Use the special **`web`** property within any style object to inject web-only CSS, including pseudo-classes (like `:hover`, `::before`). This is essential for web fidelity.
      * **Example (Hover Effect):**
        ```typescript
        image: {
          web: { ':hover': { transform: 'scale(1.1)' } }, // Web-only CSS
          transition: 'transform 0.3s ease-in-out',        // Standard RN/UniStyles property
        }
        ```
  * **Third-Party Integration:** When styling non-native React Native components (like SVG-based icons or specific third-party views), wrap them in the **`withUniStyles`** Higher-Order Component. This maps theme values to their props and enables updates without a full React re-render.
      * **Example (Icon Theming):**
        ```typescript
        import { withUniStyles } from 'react-native-unistyles';
        import { Icon } from 'lucide-react-native';

        // Icon's 'color' prop is mapped to the active theme's primary color
        export default withUniStyles(Icon, (theme) => ({
          color: theme.colors.primary,
        }));
        ```
