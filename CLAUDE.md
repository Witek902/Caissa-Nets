# Caissa-Nets

Neural network files (`.pnn`) for the Caissa chess engine (Caissa repository:
https://github.com/Witek902/Caissa). Each net is committed here and also published as a GitHub release of
https://github.com/Witek902/Caissa-Nets with the net name as tag and title and the `.pnn` as the only asset.
Caissa builds download the default net from
`https://github.com/Witek902/Caissa-Nets/releases/download/<net-name>/<net-name>.pnn`, so a released file must never
be replaced, renamed or deleted.

`HISTORY.md` is the history of all nets: architectures, a summary table (Caissa commit and release that used
each net, SPRT results) and a detail section per net. **Keep it up to date with every change listed below.**

## Adding a net

1. Copy the `.pnn` here and check it is the tested file (same hash as the training output or the net the
   SPRTs used).
2. Commit message: `Add <net-name>`, body in the style of the existing commits (`git log`): trainer commit, per
   stage the exact command and every non-default setting, data changes, `permuteNet` command, post-processing
   (SPSA tune, transplant), and the raw SPRT blocks with furybench links. Training settings are in the run's
   `<name>-settings.txt` in the trainer's output directory; results and analysis are in the Caissa repository's
   `data/knowledge/experiments-log.md` and `multilayer-net-training-experiments-log.md`.
3. Update `HISTORY.md` in the same commit: a summary table row, a `### <net-name>` section under Details, and a new
   row in the architecture table if the architecture or net format changed.
4. GitHub release: tag and title = net name, asset = the `.pnn`, notes = the commit body
   (`gh release create <net-name> <net-name>.pnn -R Witek902/Caissa-Nets --title <net-name> --notes-file <file>`).

Committing, pushing and creating releases is done manuall by the author.

## Keeping HISTORY.md current

- **Caissa switches its default net** (`NET_VERSION` in `CMakeLists.txt` and `src/makefile`): fill in "Default in
  Caissa since" with the date and Caissa commit, and the SPRT results quoted in that commit.
- **A Caissa release is tagged:** put the release in the Release column of the net it ships
  (`git show <tag>:CMakeLists.txt | grep NET_VERSION` in the Caissa repository shows the net).
- **A later test finishes** for a net already listed (LTC, endgame book, DFRC): add it to its Details section.
- Write facts with their source (commit, SPRT link); mark anything unverified instead of guessing.
