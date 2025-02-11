# X/Twitter Mass Delete Script

A JavaScript-based tool to bulk delete your X/Twitter posts. This script allows you to delete tweets from specific time periods using X's advanced search features.

## Features

- Bulk delete tweets from your timeline
- Support for advanced search queries
- Automatic scrolling and deletion
- Error handling and retry mechanisms
- Progress logging in console

## Prerequisites

- A modern web browser (Chrome, Firefox, Edge, etc.)
- Access to your X/Twitter account
- Basic knowledge of using browser developer tools

## Step-by-Step Usage Guide

### 1. Prepare Your Search Query

First, you'll need to create an advanced search query to target specific tweets. Here are some examples:

```
Basic format:
from:username until:YYYY-MM-DD since:YYYY-MM-DD

Example:
from:yourusername until:2020-01-01 since:2006-01-01
```

Advanced Search Operators:
- `from:username` - tweets from a specific account
- `until:YYYY-MM-DD` - tweets before this date
- `since:YYYY-MM-DD` - tweets after this date
- `min_faves:NUMBER` - tweets with at least NUMBER of likes
- `min_retweets:NUMBER` - tweets with at least NUMBER of retweets

### 2. Navigate to Advanced Search

1. Go to https://x.com/search
2. Enter your search query in the format described above
3. Example URL format:
   ```
   https://x.com/search?q=(from%3Ayourusername)%20until%3A2020-01-01%20since%3A2006-01-01&src=typed_query
   ```

### 3. Run the Deletion Script

1. Open your browser's Developer Tools:
   - Windows/Linux: Press `F12` or `Ctrl + Shift + I`
   - macOS: Press `Cmd + Option + I`
   
2. Navigate to the "Console" tab

3. Copy and paste the following script into the console:

```javascript
const deleteTweets = async () => {
    // Constants for selectors
    const SELECTORS = {
        moreButton: '[aria-label="More"]',
        confirmButton: '[data-testid="confirmationSheetConfirm"]'
    };

    // Helper function to wait for elements
    const waitForElement = (selector, timeout = 5000) => {
        return new Promise((resolve) => {
            if (document.querySelector(selector)) {
                return resolve(document.querySelector(selector));
            }

            const observer = new MutationObserver(() => {
                if (document.querySelector(selector)) {
                    observer.disconnect();
                    resolve(document.querySelector(selector));
                }
            });

            observer.observe(document.body, {
                childList: true,
                subtree: true
            });

            setTimeout(() => {
                observer.disconnect();
                resolve(null);
            }, timeout);
        });
    };

    // Helper function to find delete button by text content
    const findDeleteButton = () => {
        return Array.from(document.querySelectorAll('span'))
            .find(span => span.textContent === 'Delete');
    };

    // Helper function to click with retry
    const clickWithRetry = async (element, maxRetries = 3) => {
        for (let i = 0; i < maxRetries; i++) {
            try {
                element.click();
                await new Promise(resolve => setTimeout(resolve, 500)); // Wait for click to register
                return true;
            } catch (err) {
                console.log(`Click attempt ${i + 1} failed, retrying...`);
                await new Promise(resolve => setTimeout(resolve, 1000));
            }
        }
        return false;
    };

    try {
        // Find and click "More" buttons
        const moreButtons = document.querySelectorAll(SELECTORS.moreButton);
        console.log(`Found ${moreButtons.length} more buttons`);

        for (const moreButton of moreButtons) {
            // Click the "More" button
            await clickWithRetry(moreButton);
            
            // Wait a moment for the menu to appear
            await new Promise(resolve => setTimeout(resolve, 500));
            
            // Find and click "Delete" option
            const deleteButton = findDeleteButton();
            if (deleteButton) {
                await clickWithRetry(deleteButton);
                
                // Wait for and click confirm button
                const confirmButton = await waitForElement(SELECTORS.confirmButton);
                if (confirmButton) {
                    await clickWithRetry(confirmButton);
                    console.log('Tweet deleted successfully');
                }
            } else {
                // Close menu if delete option wasn't found
                document.body.click();
            }
            
            // Wait between deletions
            await new Promise(resolve => setTimeout(resolve, 1000));
        }

        // Scroll down to load more tweets
        window.scrollBy(0, 1000);
        
        // Get remaining tweets count
        const remainingTweetsElement = document.querySelectorAll('[role="heading"]+div')[1];
        if (remainingTweetsElement) {
            console.log('Remaining tweets:', remainingTweetsElement.textContent);
        }

        // Continue deletion after a delay
        setTimeout(deleteTweets, 2000);
    } catch (error) {
        console.error('Error in deletion process:', error);
        // Retry after a longer delay if there's an error
        setTimeout(deleteTweets, 5000);
    }
};

// Start the deletion process
deleteTweets();
```

### 4. Monitor the Progress

- The script will automatically:
  - Click "More" buttons (⋮)
  - Select "Delete" option
  - Confirm deletion
  - Scroll to load more tweets
  - Repeat the process

- Watch the console for progress messages:
  - Number of "More" buttons found
  - Successful deletions
  - Any errors that occur

### 5. Handling Errors

If you encounter errors:

1. Refresh the page
2. Wait a few minutes if you hit rate limits
3. Run the script again
4. Check if X's interface has been updated (selectors might need updating)

## Known Limitations

- X's rate limiting might temporarily stop the script
- Interface changes might break selectors
- Some tweets might require multiple deletion attempts
- Protected tweets and other special cases might need manual deletion

## Troubleshooting

1. **Script stops working:**
   - Refresh the page and run the script again
   - Check console for error messages
   - Verify you're logged into your account

2. **Nothing happens:**
   - Ensure you're on the correct search results page
   - Verify that tweets are visible on the page
   - Check if X's interface has been updated

3. **Rate limiting:**
   - Wait 10-15 minutes before trying again
   - Try deleting in smaller date ranges

## Safety Tips

1. **Before running:**
   - Test with a small date range first
   - Back up your data if needed
   - Verify your search query is correct

2. **During deletion:**
   - Keep the browser tab active
   - Don't interact with the page while script is running
   - Monitor the console for progress

## Contributing

Feel free to submit issues and enhancement requests!

## Disclaimer

This script is provided as-is, without warranties. Use at your own risk. Be aware that:
- Mass deletion actions are irreversible
- X's interface may change, breaking the script
- The script might need updates to match X's changes
- Always verify what you're deleting before running the script

## License

MIT License - feel free to modify and distribute as needed.

---

Remember to always review what you're deleting and make backups if necessary. This script is a tool to help automate the process, but you should use it carefully and responsibly.
