# Manage Devices with Apple Configurator
**WWDC21 · Session 10297** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10297/)

_Platforms:_ iOS 15, macOS Monterey 12

## Overview
This session covers two major areas of Apple Configurator in 2021: expanded Mac management capabilities (Restore and Revive for Apple silicon and T2 Macs) and the debut of **Apple Configurator for iPhone** — a new iOS 15 app that allows an IT administrator to assign any Mac to an organization in Apple Business Manager or Apple School Manager using just an iPhone camera, enabling Automated Device Enrollment on Macs that were not purchased through official Apple channels.

## Key Topics

**Mac Restore and Revive (Apple Configurator 2)**
Apple Configurator 2 on macOS can manage Mac computers with Apple silicon or the T2 Security Chip via a USB-C cable connected to the correct DFU port.

- *Restore*: Reinstalls firmware and the latest recoveryOS, and erases all user data. On Apple silicon, also reinstalls the latest macOS. Use to prepare a Mac for a new user or recover a fully non-functional device.
- *Revive*: Updates firmware and recoveryOS to the latest version while preserving user data. Use to recover a Mac that failed to boot (e.g., battery died mid-update) without erasing it.

The correct port varies by model: different ports are used on MacBook Air with Apple silicon, Mac mini, and MacBook Pro (T2 vs Apple silicon variants). Refer to the Configurator User Guide for port diagrams.

**Apple Configurator for iPhone (NEW — iOS 15)**
A brand-new iPhone app that enables organizations to assign any Mac (even one not purchased through Apple or an authorized reseller) to their Apple Business Manager or Apple School Manager organization. This unlocks Automated Device Enrollment for Macs acquired outside official channels.

*Workflow:*
1. Sign in to the app with a Managed Apple ID that has at least the Device Enrollment Manager role.
2. Optionally configure the Wi-Fi network the Mac should use (share the iPhone's current Wi-Fi network, or select a configuration profile with a Wi-Fi payload for non-shareable networks).
3. Boot the Mac to Setup Assistant and advance to the country/region selection screen.
4. Bring the iPhone near the Mac — an animation appears on the Mac screen.
5. Center the animation in the iPhone's camera viewfinder; the app pairs with the Mac.
6. Upon "Paired Successfully," Configurator sends the necessary enrollment data to the Mac. The Mac assigns itself to the organization.
7. In Apple Business Manager or Apple School Manager, move the newly assigned Mac from the Apple Configurator 2 MDM server to the desired MDM server.
8. On next restart, the Mac downloads Automated Device Enrollment settings and self-configures.

*30-Day Provisional Period:* A Mac assigned via this feature can be released from the organization if the user unenrolls from MDM within 30 days (matching iOS behavior).

**Automated Device Enrollment + AutoAdvance**
When combined with AutoAdvance, Automated Device Enrollment enables completely touch-free Mac setup: connect to power and Ethernet, and the Mac self-enrolls and self-configures with no user interaction.

**Status View**
The iPhone app includes a Status view showing nearby Macs that can be assigned and those already assigned, giving visibility into assignment operations in progress.

## APIs & Frameworks

- **Apple Configurator for iPhone** **[NEW]** — iOS 15 app (App Store, fall 2021); requires Managed Apple ID with Device Enrollment Manager role
  - Camera-based Mac pairing via on-screen animation in Setup Assistant
  - Wi-Fi sharing or configuration profile for network provisioning
  - Status view for assignment tracking
- **Apple Configurator 2** (macOS) — Mac Restore and Revive **[UPDATED]**
  - Restore action — firmware reinstall + erase (Apple silicon: also reinstalls macOS)
  - Revive action — firmware update, user data preserved
  - Supported hardware: Apple silicon Macs, Mac with T2 Security Chip
- **Automated Device Enrollment (ADE)** — MDM enrollment flow integrated into Setup Assistant
- **AutoAdvance** — touch-free Mac provisioning over Ethernet
- **Apple Business Manager / Apple School Manager** — device assignment and MDM server targeting
- **MDM (Mobile Device Management)** — configuration delivery post-enrollment
- Configuration Profile (`.mobileconfig`) with Wi-Fi payload — optional network provisioning via Configurator for iPhone
- Managed Apple ID — authentication for Device Enrollment Manager role

## Code Highlights

No Swift/Objective-C developer API introduced in this session. The session covers an IT operations workflow implemented entirely through the Apple Configurator for iPhone app, Apple Configurator 2, and Apple Business Manager/Apple School Manager web portals.

## Takeaways

- Apple Configurator for iPhone removes the last barrier to Automated Device Enrollment: Macs purchased outside official channels can now be enrolled in ABM/ASM using only an iPhone camera and a Managed Apple ID — no cable, no Mac-side software.
- Use Restore (erases) vs. Revive (preserves data) to match the recovery scenario; both require connecting to the correct DFU port on the Mac with a USB-C cable.
- The 30-day provisional period means organizations should monitor for unenrollment events within the first month after assigning a Mac via Apple Configurator for iPhone.
- AutoAdvance + ADE enables fully zero-touch Mac provisioning: power + Ethernet is all a device needs to self-configure to an organization's specifications.

---
_Source: WWDC21 Session 10297 page (abstract, full transcript, and resource links)._
