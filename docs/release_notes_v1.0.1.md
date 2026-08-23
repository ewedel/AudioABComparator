# Audio A/B Comparator 1.0.1

Audio A/B Comparator 1.0.1 fixes waveform alignment when the two loaded tracks have different total durations.

## Fixed

- both waveforms are now rendered over the same common playback duration;
- matching audio events no longer receive an artificial horizontal offset from independent waveform stretching;
- timing differences actually present in the audio remain visible and audible;
- no normalization, time stretching or automatic realignment is applied.

## Validation

- Linux x86_64 AppImage built under Ubuntu 22.04 with Qt 6.9.3;
- Windows 10/11 x64 portable ZIP built natively with MSVC and Qt 6.9.3;
- complete Linux and Windows test suites passed;
- packaged smoke tests passed in English, French, German, Spanish, Brazilian Portuguese, Japanese and Simplified Chinese.

The Windows executable is unsigned and may trigger a SmartScreen warning. Verify both downloads with `SHA256SUMS`.

---

Audio A/B Comparator 1.0.1 corrige l’alignement des formes d’onde lorsque les deux pistes chargées n’ont pas exactement la même durée totale.

## Correction

- les deux formes d’onde sont désormais affichées sur la même durée commune de lecture ;
- les événements audio correspondants ne subissent plus de décalage horizontal artificiel dû à un étirement indépendant ;
- les différences temporelles réellement présentes dans l’audio restent visibles et audibles ;
- aucune normalisation, aucun étirement temporel et aucun recalage automatique ne sont appliqués.

## Validation

- AppImage Linux x86_64 construite sous Ubuntu 22.04 avec Qt 6.9.3 ;
- ZIP portable Windows 10/11 x64 construit nativement avec MSVC et Qt 6.9.3 ;
- suites de tests Linux et Windows entièrement réussies ;
- smoke tests des paquets réussis en anglais, français, allemand, espagnol, portugais brésilien, japonais et chinois simplifié.

L’exécutable Windows n’est pas signé et peut déclencher un avertissement SmartScreen. Vérifiez les deux téléchargements avec `SHA256SUMS`.
