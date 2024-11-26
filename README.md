# Uso de Threads no projeto opensource Runelite:

## O que é o Runelite?
O Runelite é um projeto open-source em Java, que funciona como um ***client para o jogo Old School RuneScape***. O projeto tem como objetivo melhorar a experiência dos jogadores, trazendo uma interface com opções de personalização e instalação de plugins que facilitem e garantam uma melhor _qualidade de vida_ dentro do jogo.

### Por que escolhi pesquisar sobre o Runelite?
Após ser mencionado em aula o projeto do RoboCode, pensei em procurar sobre usos similhares - em jogos; lembrei-me sobre o uso do Runelite na época que jogava OSRS e optei por pesquisar um pouco sobre como foi desenvolvido e como funcionaria o uso de threads em um projeto de grande escala.

## Análise de código:
Por se tratar de um projeto muito grande, com um quantidade asssustadora de arqvuios e linhas de código, separei pequenos trechos onde foram usadas threads.
### Carregar imagens parcialmente:
```

	public static BufferedImage create(File inputFile, int imageType) throws IOException
	{
		try (ImageInputStream stream = ImageIO.createImageInputStream(inputFile))
		{
			Iterator<ImageReader> readers = ImageIO.getImageReaders(stream);
			if (readers.hasNext())
			{
				try
				{
					ImageReader reader = readers.next();
					reader.setInput(stream, true, true);
					int width = reader.getWidth(reader.getMinIndex());
					int height = reader.getHeight(reader.getMinIndex());
					BufferedImage image = create(width, height, imageType);
					int cores = Math.max(1, Runtime.getRuntime().availableProcessors() / 2);
					int block = Math.min(MAX_PIXELS_IN_MEMORY / cores / width, (int) (Math.ceil(height / (double) cores)));
					ExecutorService generalExecutor = Executors.newFixedThreadPool(cores);
					List<Callable<ImagePartLoader>> partLoaders = new ArrayList<>();
					for (int y = 0; y < height; y += block)
					{
						partLoaders.add(new ImagePartLoader(
							y, width, Math.min(block, height - y), inputFile, image));
					}
					generalExecutor.invokeAll(partLoaders);
					generalExecutor.shutdown();
					return image;
				}
				catch (InterruptedException ex)
				{
					log.error(null, ex);
				}
			}
		}
		return null;
	}
```

### Trecho que realmente nos importa: 
```
int cores = Math.max(1, Runtime.getRuntime().availableProcessors() / 2);
int block = Math.min(MAX_PIXELS_IN_MEMORY / cores / width, (int) (Math.ceil(height / (double) cores)));
ExecutorService generalExecutor = Executors.newFixedThreadPool(cores);
List<Callable<ImagePartLoader>> partLoaders = new ArrayList<>();

```
**ExecutorService**: é uma interface do pacote `java.until.concurrent.` Facilita a vida para criar, finalizar e gerenciar threads.

   - Nesse caso, foi utilizado para criar uma ***pool com um número fixo de threads*** a serem usadas.
   - Thread pools: são threads reutilizaveis pelo ExecutorService. Se todas as threads estiverem ocupadas, a tarefa é colocada em um queue.
   - Foi usado para ***paralelizar*** a tarefa de carregar as partes de uma imagem. _Isso faz com que o processo seja mais eficiente._
  
## Análise da classe ImagePartLoader:

```
private static class ImagePartLoader implements Callable<ImagePartLoader> {
    private final int y;

    public ImagePartLoader call() throws Exception {
        Thread.currentThread().setPriority((Thread.MIN_PRIORITY + Thread.NORM_PRIORITY) / 2);
        try (ImageInputStream stream = ImageIO.createImageInputStream(file);) {
            // Image processing logic
        }
    }
}
```
**Callable**: é uma interface do ExecutorService. É usada para implementar o método call.(semelhante ao Runnable que vimos em aula).
  - `Call()`: executa uma tarefa e ajusta a prioridade da thread. Também utiliza paralelismo.

### Projeto completo: [Runelite Github](https://github.com/runelite/runelite).
##
### Obrigado!
##

**Referências:**
- [Threads Java - Geeks for geeks](https://www.geeksforgeeks.org/java-threads/)
- [ExecutorService - Geeks for geeks](https://www.geeksforgeeks.org/java-util-concurrent-executorservice-interface-with-examples/)
- [Video sobre Threads](https://www.youtube.com/watch?v=xNBMNKjpJzM)
- *uso de IA para identificação de código no projeto.
