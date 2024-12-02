<script setup lang="ts">
import { Info } from 'lucide-vue-next';
import { h } from 'vue';

import {
  Select,
  SelectContent,
  SelectGroup,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select';
import { useToast } from '@/components/ui/toast';
import { commands } from '@/services/bindings';

import DownloadDialog from './DownloadDialog.vue';

const { data: sounds, refresh: refreshSounds } = useRequest(async () => {
  const res = await commands.getSounds();
  if (res.status === 'ok') {
    return res.data ?? [];
  }
});

const { data: selectedSound, refresh: refreshSelectedSound } = useRequest(
  async () => {
    const res = await commands.getSelectedSound();
    if (res.status === 'ok') {
      return res.data ?? '';
    }
  },
);

const { toast } = useToast();

const { run: selectSound } = useRequest(
  async (sound: string) => {
    const res = await commands.selectSound(sound);
    if (res.status === 'error') {
      toast({
        variant: 'destructive',
        duration: 10000,
        description: h('div', { class: 'flex items-center gap-2' }, [
          h(
            'span',
            "Your selected sound needs an update. Please re-download it. If re-downloading the sound didn't resolve the problem, click on the info icon to the right for further instructions.",
          ),
          h(
            'button',
            {
              class: 'p-1 rounded hover:bg-secondary',
              onClick: () => {
                toast({
                  description: `To resolve this issue, you need to:
1. Go to ~/Library/Application Support/xyz.waveapps.keyecho/sounds
2. Create a folder named according to the sound name
3. Copy sound.ogg and config.json into the newly created folder
4. Go back to KeyEcho and it should work now.`,
                  duration: 10000,
                });
              },
            },
            [h(Info, { class: 'w-4 h-4' })],
          ),
        ]),
      });
    } else {
      const soundItem = sounds.value?.find((s) => s.value === sound);
      if (soundItem) {
        toast({
          duration: 1000,
          description: `'${soundItem.name}' chosen successfully.`,
        });
      }
    }
  },
  {
    manual: true,
    onAfter: () => {
      refreshSelectedSound();
    },
  },
);

const existedSoundNames = computed(() =>
  (sounds.value ?? []).map((s) => s.name),
);
const hasExistedSounds = computed(() =>
  Boolean(existedSoundNames.value.length),
);
</script>

<template>
  <div>
    <Select v-model="selectedSound" @update:model-value="selectSound">
      <SelectTrigger :disabled="!hasExistedSounds">
        <SelectValue
          :placeholder="
            hasExistedSounds ? 'Select Your Sound' : 'Download Sounds to Select'
          "
        />
      </SelectTrigger>
      <SelectContent class="max-h-[42vh]">
        <SelectGroup>
          <SelectItem v-for="s in sounds" :key="s.value" :value="s.value">
            <span class="whitespace-nowrap">{{ s.name }}</span>
          </SelectItem>
        </SelectGroup>
      </SelectContent>
    </Select>

    <DownloadDialog
      :refreshSounds="refreshSounds"
      :existedSoundNames="existedSoundNames"
    />
  </div>
</template>
