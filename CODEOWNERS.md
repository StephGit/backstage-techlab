# In der CODEOWNERS-Datei können Kommentare geschrieben werden, wie dieser hier.
# Jede Zeile ohne '#' ist ein File-Pattern gefolgt von einer Liste von Owners.
# Wenn man sich an die Konventionen von GitHub hält, kann GH da direkt einige Features integrieren.
# Für uns ist aber wichtiger, dass wir wissen, wer bei Puzzle zuständig ist.
# GitHub hat eine ausführliche Dokumentation, was alles möglich ist. Wir setzen hier nur das Minimum.


# Damit werden zwei globale Owners für das Repo gesetzt.
# Sie werden z.B. bei PRs auch angeschrieben.
# Die Codeowner werden über ihre Puzzle-Email identifiziert.
# Damit ist es einfacher zu bestimmen, wer gemeint ist.
# Mehrere Codeowner können in einer Liste referenziert werden:
*       raaflaub@puzzle.ch girod@puzzle.ch

# Soll für ein Teil des Repos spezifisch ein anderer Owner gesetzt werden,
# geht das auch, und spätere Matches überschreiben vorherige Matches.
# Hier wird für /example ein neuer Owner gesetzt, aber für /example/specific
# gelten wieder die globalen Owner, da dort die Owner-Liste leer ist.
/example example@puzzle.ch
