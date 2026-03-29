---
name: trae-control
description: Control Trae AI IDE via CDP (Chrome DevTools Protocol). Use browser tools to interact with Trae's chat interface.
---

# Trae Control Skill

Control Trae AI IDE desktop application through its built-in CDP (Chrome DevTools Protocol) port.

## Connection

- **CDP Port**: 9222 (default)
- **WebSocket URL**: `ws://localhost:9222/devtools/page/<target-id>`
- **Browser Debugger**: `http://localhost:9222/json`

## Trae UI Selectors

| Element | Selector | Method |
|---------|----------|--------|
| Chat Input | `.chat-input-v2-input-box-editable` | `page.keyboard.type()` |
| Send Button | `.chat-input-v2-send-button` | `page.click()` |
| AI Response | `[data-role="assistant"]` | `querySelectorAll()` |

## Input Method (Critical)

**Use `page.keyboard.type()` - this is the working method!**

1. Click on input element
2. Use `keyboard.type()` to type text (not `keyboard.sendCharacter()` or direct textContent manipulation)
3. Click send button

```javascript
// ✅ Working method
await page.click('.chat-input-v2-input-box-editable');
await page.keyboard.type('Hello Trae!');
await page.click('.chat-input-v2-send-button');

// ❌ Does NOT work
await page.evaluate(el => el.textContent = 'text', inputElement);
```

## Workflow Examples

### Basic Chat

```
workflow:
  - name: Send message to Trae
    steps:
      - connect_cdp:
          port: 9222
          browserURL: "http://localhost:9222"
      - find_element:
          selector: ".chat-input-v2-input-box-editable"
      - click_element
      - type_text:
          text: "{user_message}"
          method: keyboard.type
      - click:
          selector: ".chat-input-v2-send-button"
      - wait_for_response:
          selector: "[data-role=\"assistant\"]"
          timeout: 60000
```

### Wait for AI Response

```javascript
// Get latest AI response
const response = await page.evaluate(() => {
  const messages = document.querySelectorAll('[data-role="assistant"]');
  if (messages.length > 0) {
    return messages[messages.length - 1].textContent.trim();
  }
  return null;
});
```

## Tips

1. **Always wait** after clicking send (AI response takes 1-5 seconds)
2. **Input method**: MUST use `page.keyboard.type()` - other methods don't work
3. **Network errors**: Check if Trae is connected to internet
4. **Selector changes**: If selectors don't work, check Trae version

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Cannot connect | Check if Trae is running with CDP enabled (port 9222) |
| Input not appearing | Use `keyboard.type()` instead of direct manipulation |
| No send button | Press Enter as fallback |
| No response | Wait longer (up to 60s) or check network |

## Technical Notes

- Trae is an Electron app based on Chromium
- CDP is built into Electron, no extra installation needed
- Use `browserURL` connection method: `puppeteer.connect({ browserURL: 'http://localhost:9222' })`
- Puppeteer-core is used (no bundled Chromium needed)
