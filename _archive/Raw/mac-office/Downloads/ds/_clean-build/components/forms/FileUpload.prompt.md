```jsx
<FileUpload multiple accept=".pdf,.dxf,.step"
  hint="PDF, DXF or STEP. 25 MB a file."
  files={[
    { name: 'bracket-rev-c.step', size: '4.2 MB' },
    { name: 'assembly.pdf', size: '18 MB', progress: 62 },
    { name: 'notes.docx', size: '1.1 MB', error: 'That file type is not accepted — use PDF, DXF or STEP.' }
  ]}
  onFiles={add} onRemove={drop} />
```

State the limits in `hint` **before** someone hits them. Errors go on the row that failed and say what to do — never a single "Upload failed" for a batch. The drop zone uses a dashed border, deliberately not the dotted signature edge: dashed reads as provisional, which is what a drop target is.
