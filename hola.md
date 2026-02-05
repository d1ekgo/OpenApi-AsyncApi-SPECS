Run PROMPT="OpenApi-AsyncApi-SPECS/prompts/openapi-prompt.md"
  PROMPT="OpenApi-AsyncApi-SPECS/prompts/openapi-prompt.md"
  
  # Usa modelo Codex (más orientado a código) y evita auto-update del CLI.
  # Flags oficiales: --model, --no-auto-update, -p/--prompt, --allow-all-tools, -s/--silent, --stream off
  copilot \
    --no-auto-update \
    --model "GPT-5.2-Codex" \
    -p "$(cat "$PROMPT")" \
    --allow-all-tools \
    --disable-parallel-tools-execution \
    -s \
    --stream off \
    > copilot.md
  
  echo "🔎 Copilot raw output:"
  cat copilot.md
  
  COPILOT_ERRORS=$(grep "copilotErrors:" copilot.md | head -1 | sed 's/[^0-9]*//g')
  COPILOT_WARNINGS=$(grep "copilotWarnings:" copilot.md | head -1 | sed 's/[^0-9]*//g')
  if [ -z "$COPILOT_ERRORS" ]; then COPILOT_ERRORS=0; fi
  if [ -z "$COPILOT_WARNINGS" ]; then COPILOT_WARNINGS=0; fi
  
  COPILOT_RULES=$(grep "\[Regla Violada\]" copilot.md | cut -d':' -f2 | sed 's/^ *//' | tr '\n' ',' | sed 's/,$//')
  
  echo "copilot_errors=$COPILOT_ERRORS" >> $GITHUB_OUTPUT
  echo "copilot_warnings=$COPILOT_WARNINGS" >> $GITHUB_OUTPUT
  echo "copilot_rules=$COPILOT_RULES" >> $GITHUB_OUTPUT
  shell: /usr/bin/bash -e {0}
  env:
    GH_TOKEN: ***
    COPILOT_GITHUB_TOKEN: ***
  
error: option '--model <model>' argument 'GPT-5.2-Codex' is invalid. Allowed choices are claude-sonnet-4.5, claude-haiku-4.5, claude-opus-4.5, claude-sonnet-4, gemini-3-pro-preview, gpt-5.2-codex, gpt-5.2, gpt-5.1-codex-max, gpt-5.1-codex, gpt-5.1, gpt-5, gpt-5.1-codex-mini, gpt-5-mini, gpt-4.1.
Error: Process completed with exit code 1.
