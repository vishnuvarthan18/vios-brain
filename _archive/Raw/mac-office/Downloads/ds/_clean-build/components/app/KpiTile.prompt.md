```jsx
<CardGrid width="narrow">
  <KpiTile label="Open jobs" value="48" delta="+6 this week" direction="up" />
  <KpiTile label="Overdue" value="3" delta="-2" direction="down" note="Down from 5" />
</CardGrid>
```

The delta carries an arrow as well as a colour — roughly one man in twelve cannot reliably tell the green from the red. Up is not always good: an "Overdue" figure going up is bad, so `direction` describes the movement and you choose which colour that deserves.
