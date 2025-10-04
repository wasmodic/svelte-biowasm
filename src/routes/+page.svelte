<script lang="ts">
  import Aioli from "@biowasm/aioli";
  import { Switch } from '@skeletonlabs/skeleton-svelte';

  let inputSequence = $state('');
  let outputFormat = $state('Clustal');
  let isProcessing = $state(false);
  
  // Dark mode toggle state
  let checked = $state(false);

  $effect(() => {
    const mode = localStorage.getItem('mode') || 'light';
    checked = mode === 'dark';
  });

  const onCheckedChange = (event: { checked: boolean }) => {
    const mode = event.checked ? 'dark' : 'light';
    document.documentElement.setAttribute('data-mode', mode);
    localStorage.setItem('mode', mode);
    checked = event.checked;
  };

  

  // Example sequence
  const exampleSequence: string = `>test1 
MGDVEKGKKIFIMKCSQCHTVEKGGKHKTGPNLHGLFGRKTGQAPGYSYTAANKNKGIIW 
GEDTLMEYLENPKKYIPGTKMIFVGIKKKEERADLIAYLKKATNE  
>test2 
MGDVEKGKKIFVQKCAQCHTVEKGGKHKTGPNLHGLFGRKTGQAAGFSYTDANKNKGITW 
GEDTLMEYLENPKKYIPGTKMIFAGIKKKGERADLIAYLKKATNE 
>test3 
MTEFKAGSAKKGATLFKTRCLQCHTVEKGGPHKVGPNLHGIFGRHSGQAEGYSYTDANIK 
KNVLWDENNMSEYLTNPKKYIPGTKMAFGGLKKEKDRNDLITYLK`;

  // MUSCLE alignment function with format support
  async function performAlignment(sequence: string, format: string): Promise<string> {
    const CLI = await new Aioli(["muscle/3.8.1551"]);
    
    await CLI.mount({
      name: "input.fa",
      data: sequence
    });
    
    // Configure format flags and file extensions
    const formatConfig: Record<string, { flag: string; ext: string }> = {
      'FASTA': { flag: '', ext: 'fa' },
      'Clustal': { flag: '-clw', ext: 'aln' },
      'Clustal (strict)': { flag: '-clwstrict', ext: 'aln' },
      'HTML': { flag: '-html', ext: 'html' },
      'GCG MSF': { flag: '-msf', ext: 'msf' },
      'Phylip interleaved': { flag: '-phyi', ext: 'phy' },
      'Phylip sequential': { flag: '-phys', ext: 'phy' }
    };
    
    const config = formatConfig[format] || formatConfig['FASTA'];
    const outputFile = `output.${config.ext}`;
    let muscleCommand = `muscle -in input.fa -out ${outputFile}`.trim();
    if (config.ext !== 'fa') {
      muscleCommand = `muscle ${config.flag} -in input.fa -out ${outputFile}`.trim();
    }
    await CLI.exec(muscleCommand);
    const output = await CLI.cat(outputFile);
    return output;
  }

  // Download file function
  function downloadFile(content: string, filename: string): void {
    const blob = new Blob([content], { type: 'text/plain' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = filename;
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
  }

  // Form submit handler
  async function handleSubmit(event: Event): Promise<void> {
    event.preventDefault();
    
    if (!inputSequence.trim()) {
      alert('Please enter a sequence');
      return;
    }

    isProcessing = true;
    
    try {
      const alignedResult = await performAlignment(inputSequence, outputFormat);
      const filename = `alignment.${outputFormat.toLowerCase()}`;
      downloadFile(alignedResult, filename);
    } catch (error) {
      console.error('Alignment failed:', error);
      alert('Alignment failed. Please try again.');
    } finally {
      isProcessing = false;
    }
  }

  // Use example sequence
  function useExample(): void {
    inputSequence = exampleSequence;
  }

  // Clear sequence
  function clearSequence(): void {
    inputSequence = '';
  }

  // File upload handler
  function handleFileUpload(): void {
    const input = document.createElement('input');
    input.type = 'file';
    input.accept = '.fasta,.fa,.fas,.txt,.seq';
    
    input.onchange = (event: Event) => {
      const target = event.target as HTMLInputElement;
      const file = target.files?.[0];
      
      if (file) {
        const reader = new FileReader();
        reader.onload = (e: ProgressEvent<FileReader>) => {
          const content = e.target?.result as string;
          if (content) {
            inputSequence = content;
          }
        };
        reader.readAsText(file);
      }
    };
    
    input.click();
  }
</script>

<svelte:head>
  <script>
    const mode = localStorage.getItem('mode') || 'light';
    document.documentElement.setAttribute('data-mode', mode);
  </script>
</svelte:head>

<div class="flex justify-between items-center mb-4">
  <h1 class="h1">Muscle WebAssembly</h1>
  <Switch {checked} {onCheckedChange}></Switch>
</div>
<p class="mb-6 text-gray-700 dark:text-gray-300">
  MUSCLE (MUltiple Sequence Comparison by Log-Expectation) running entirely in your browser via WebAssembly. 
  This client-side implementation provides fast, accurate multiple sequence alignment without requiring server uploads - 
  your data never leaves your computer.
</p>
<form class="mx-auto w-full space-y-4" onsubmit={handleSubmit}>
  <label class="label">
    <span class="label-text h4">Input Sequence</span>
    <textarea
      bind:value={inputSequence}
      class="textarea"
      rows="4"
      placeholder="Paste your sequence here - or use the example sequence"
    ></textarea>
  </label>
  <button type="button" class="btn preset-filled-primary-500" onclick={handleFileUpload}>Upload File</button>
  <div class="btn-group w-full justify-end">
      <button type="button" class="btn preset-filled-secondary-500" onclick={useExample}>Use the example</button>
      <button type="button" class="btn preset-outlined-secondary-500" onclick={clearSequence}>Clear sequence</button>
  </div>

  <div class="form-control">
    <label class="label cursor-pointer">
      <span class="label-text h4">Output Format</span>
      <select bind:value={outputFormat} class="select select-bordered w-full max-w-xs">
        <option>FASTA</option>
        <option>Clustal</option>
        <option>Clustal (strict)</option>
        <option>HTML</option>
        <option>GCG MSF</option>
        <option>Phylip interleaved</option>
        <option>Phylip sequential</option>
      </select>
    </label>
  </div>
  <button 
    type="submit" 
    class="btn preset-filled-primary-500 w-full"
    disabled={isProcessing}
  >
    {isProcessing ? 'Processing...' : 'Run Alignment'}
  </button>
</form>
<div class="text-center mt-6">
    <a class="mt-4 text-sm text-gray-500 dark:text-gray-400 underline" href="https://github.com/Wytamma/embl-ebi-muscle-wasm">
        Open source code on GitHub</a>
</div>
