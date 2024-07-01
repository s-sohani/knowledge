While developing system, some module may update, this update include code or database schema.  
In order for the system to continue running smoothly while updating, we need to maintain compatibility in both directions:
- Backward compatibility :Newer code can read data that was written by older code.
- Forward compatibility: Older code can read data that was written by newer code.
Backward compatibility is easy when writing new version of cod