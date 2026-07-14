# Accessibility (a11y) Guidelines for Custom Modal Interfaces

This document outlines the accessibility standards and best practices required when building custom modal interfaces (dialog boxes) and custom selectors within the WorkSphere platform.

Adhering to these guidelines ensures our application is fully usable for individuals relying on screen readers and keyboard navigation.

---

## 1. Screen-Reader Tags & ARIA States

Custom modals must clearly communicate their purpose and state to assistive technologies.

- **`role="dialog"` or `role="alertdialog"`:** The container element of the modal must have this role. Use `alertdialog` only when the modal requires immediate user attention (e.g., a destructive action warning).
- **`aria-modal="true"`:** This attribute must be added to the modal container. It tells assistive technologies that the content outside the modal is currently unavailable and should be ignored.
- **`aria-labelledby`:** The modal container should use this attribute to point to the `id` of the modal's visible title element.
- **`aria-describedby` (Optional):** If the modal has a descriptive text block, use this to point to the `id` of that text element.

## 2. Trigger Elements and `aria-expanded`

The element (usually a button) that opens the modal or custom selector must manage its state so screen readers know if the component is open or closed.

- Add `aria-expanded="false"` to the trigger button by default.
- Toggle it to `aria-expanded="true"` when the modal or selector is opened.
- Ensure the trigger element also has `aria-haspopup="dialog"` (or `listbox` for custom selectors).

## 3. Keyboard Focus Management (The "Focus Trap")

Modals must strictly manage user focus to prevent them from interacting with the background page while the modal is open.

- **Initial Focus:** When the modal opens, focus must automatically move to the first focusable element inside the modal (or the close button, depending on context).
- **Focus Trapping:** While open, pressing `Tab` must cycle focus forward through the modal's interactive elements, and `Shift + Tab` must cycle backward. Focus **must never** escape the modal container.
- **Closing via Keyboard:** Pressing the `Escape` key must close the modal.
- **Restoring Focus:** When the modal closes, focus must immediately return to the exact element that originally triggered the modal.

## 4. Overlays (Backdrops) and Custom Selectors

The visual overlay that sits behind the modal and covers the rest of the application must also be handled correctly.

- **Click-to-Close:** Clicking on the overlay backdrop should close the modal or custom selector.
- **Hidden from Screen Readers:** The overlay itself should not be focusable and should ideally be hidden from screen readers using `aria-hidden="true"` (if it is a separate element from the modal container).
- **Scroll Locking:** When the overlay is active, the main `<body>` of the document should have its scroll locked (e.g., `overflow: hidden`) to prevent background scrolling.

---

### Quick Implementation Checklist

- [ ] Modal container has `role="dialog"` and `aria-modal="true"`.
- [ ] Trigger button toggles `aria-expanded` between `true` and `false`.
- [ ] `Escape` key closes the modal.
- [ ] `Tab` cycles focus only within the modal (focus trap).
- [ ] Focus returns to the trigger button upon closing.
- [ ] Clicking the background overlay closes the modal.
