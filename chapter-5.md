# CHAPTER 5 CONCLUSIONS AND RECOMMENDATIONS

## 5.1 Conclusions
This project demonstrates technologies and features that can be considered for hybrid meeting solutions. The project covers the backend design and implementation for the hybrid meeting platform HereToo.

This project utilizes FastAPI, a Python framework for creating web applications as the backbone of the platform. FastAPI is used to define API routes and manage databases that can be accessed by the frontend. To ensure a seamless connectivity between participants WebSocket technology is adopted. This technologies are implemented so that the host can manage the flow of the platform and the participants can engage with each other. Participants will be playing a game of bingo where they can meet other participants with the same interest ensuring they have something in common as a foundation to interact. Having the participants in a relaxed environment by playing games can build a strong connection even in a hybrid setting.

## 5.2 Recommendations
- This project only covers the backend design and implementation thus still requires an extensive research for the frontend side of the platform.
- Tests that scales the platform must be conducted as this project only covers an internal test with a small subset of participants.
- Security mechanism must be enhanced especially since the platform is integrated with Zoom Application. Proper authentication and authorization must be implemented to ensure participants safety.
