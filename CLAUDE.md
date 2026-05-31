# The Intern

"Imagine going to live on a mountaintop by yourself, forever. You build a home that no one will ever visit. Still, you invest the time and effort to shape the space in which you’ll spend your days. The wood, the plates, the pillows—all magnificent. Curated to your taste.”
- Rick Rubin

## Summary
This project is my digital mountaintop home. It is a summary of my personal Intern. But I've stripped out the personal details. Which in many ways takes the "soul" of the intern away. So please consider this project the "sharable blueprints" of the mountaintop home. But the real house, my real intern, you will never see nor experience.

## Components or Constraints
- Memory stored locally, but backed up to git repository. Leverage [mempalace](https://github.com/mempalace/mempalace)
- Projects are stored in a predictable structure. Can be deleted because we have them in git history. There are Intern projects and work projects. The difference is file path. The intern stores projects in: ~/p/master/projects. NOTE: ~/p is a symlink to ~/projects in my house. "work projects" (like my applications) are in ~/p. For example: a standalone app I'm building lives at ~/p/myapp, while a trip the intern is planning for me is stored in ~/p/master/projects/my-trip. See the difference? The Intern's CLAUDE.md *can* interact with projects in ~/p/ if and only if I tell it to.
- Agent and model agnostic. Switch from Gemini to Claude to Gemma. Absolutely no ties. Just a CLI agent tool.
- Multiple front-doors. Text the Intern via iMessage. Have a web interface like [tmux-terminal](https://github.com/xeb/tmux-terminal).
- Only me, the owner of the house, uses the tmux terminal. Everyone else in the family only uses iMessage.
- Rooms of the house are **tmux windows**. Everything depends on tmux and tmux naming.
- Personalized scripts. Expect to have a lot of personalized scripts.
- Leverage CLAUDE.md for personalized instructions and authorization. 
- GEMINI.md is a symlink to CLAUDE.md
- Daily questions are used to "fill in memory over time" such as: "Where did you grow up?" These get stored in the memory files.
- contacts.toml is used by the iMessage interface to manage user identification. And authorization is done by Claude via CLAUDE.md
- [sink](https://github.com/xeb/sink) is used as the iMessage process and [BlueBubbles](https://bluebubbles.app/) is used on my Mac mini as the API
