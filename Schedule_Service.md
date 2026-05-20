# Scheduling Feature UI Implementation

This plan outlines the frontend UI implementation for the new Scheduling feature based on the provided mockup images. As requested, we will focus exclusively on the UI structure and interactions, ignoring backend server logic and device group notions for now.

## Proposed Changes

We will implement two distinct UI components using existing libraries in your stack (`date-fns` for calendar logic and `react-day-picker` for date picking).

### 1. Publish Schedule Modal

We will replace the existing direct-publish modal in the Playlist Editor with the new Schedule Publish Modal.

#### [NEW] `PublishScheduleModal.tsx`
Location: `plaisoram_web/src/app/(dashboard)/playlists/editLayout/components/PublishScheduleModal.tsx`

- A centered popup modal matching the "Save content & Publish to your device" design.
- **Form Controls**:
  - **Target Device**: A standard HTML `<select>` or Radix dropdown to pick the destination screen (replacing the "Device Group" notion per your instruction).
  - **Publish Time**: Custom styled radio buttons for `Immediately` and `Custom`.
  - **Date Picker**: When `Custom` is selected, an input field "Select date & time" will appear with a calendar icon. Clicking it will open a calendar popover (using your existing `react-day-picker` dependency).
- **Actions**:
  - `Back` button to cancel.
  - `Publish` blue primary button to confirm. For now, this will trigger a success toast and close the modal without making a real API request.

#### [MODIFY] `page.tsx` (Playlist Editor)
Location: `plaisoram_web/src/app/(dashboard)/playlists/editLayout/page.tsx`
- Replace the old `DevicePickerModal` component with the new `PublishScheduleModal`.
- Wire up the "Save & Publish" button to open this new modal.

---

### 2. Full Schedules Calendar View

We will implement a new dedicated page for the interactive Calendar.

#### [NEW] `page.tsx` (Schedules Page)
Location: `plaisoram_web/src/app/(dashboard)/schedules/page.tsx`

- **Header / Toolbar**:
  - Breadcrumb: `Schedules | calendar`.
  - Dropdown filter: `Default Group`.
  - Navigation controls: `<` (previous), `>` (next), and `today` buttons.
  - Title: Centered month string (e.g., `MARCH 2026`).
  - View Toggles: `month`, `week`, `day` segmented control.
- **Calendar Grid Layout**:
  - A responsive 7-column CSS grid displaying the days of the week (`Sun`, `Mon`, `Tue`...).
  - Dynamic generation of calendar cells using `date-fns` to fill the 5 or 6 rows needed for the selected month.
  - Clean, border-separated white cell design matching the mockup precisely.
  - Empty day cells ready for future schedule event blocks.
