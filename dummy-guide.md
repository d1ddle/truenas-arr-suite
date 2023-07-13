TrueNAS Arr Suite and Media Centre guide for dummies. Created 13/7/2023 for TrueNAS scale 22.12.3.1
Section 1 - Install
- Setup hardware - ideally one SSD as boot disk and as many HDDs as data storage. Minimum 1, Recommended 2 the same size for [RAID mirroring](https://www.techtarget.com/searchstorage/definition/disk-mirroring).
- Flash [TrueNAS Scale ISO](https://www.truenas.com/download-truenas-scale/) to a USB drive using [rufus](https://rufus.ie/en/):
  - **This will erase all data on the USB. Backup before continuing.**
  - Select USB drive. Check `List USB hard drives` if you can't find yours. **Click Start.**
  - Check flash in ISO Mode (or the Recommended setting) if a Hybrid ISO prompt appears
- Boot system from the TrueNAS USB:
